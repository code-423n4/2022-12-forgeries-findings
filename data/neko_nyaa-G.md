### Gas optimizations list

| Number |Details|Context|Approx. gas saved|
|:--:|:-------|:--:|:--:|
|[G-01]| Use smaller data types to better pack storage | 2 | 63000 |
|[G-02]| Check all parameters before writing into storage | 2 | 84000 - 189000 on revert|
|[G-03]| Cache storage variables into memory to avoid re-reading | 3 | 97 - 388 |
|[G-04]| Perform all operations before emitting event | 4 | 2100 - 18900 on revert|
|[G-05]| Redundant condition check | 2 | 200 |
|[G-06]| Repeated information in emitted event | 1 | 200 |
|[G-07]|`redraw`: It is possible to trick `_requestRoll` into rerolling with much less gas cost | 1 | 42000 |
|[G-08]|`InitializedDraw`: Don't use storage access for event | 1 | 873 |

## [G-01] Use smaller data types to better pack storage

Keeping functionality and code organization in find, we recommend the following storage layout for `Settings` and `CurrentRequest` struct for gas saving:

- For `Settings`:
```solidity=
struct Settings 
    address token;
    uint256 tokenId;
    address drawingToken;
    uint256 drawingTokenStartId;
    uint256 drawingTokenEndId;
    uint256 drawBufferTime; /// @audit one slot
    uint256 recoverTimelock; /// @audit one slot
    bytes32 keyHash;
    uint64 subscriptionId; /// @audit one slot
}
```
Change to
```solidity=
struct Settings 
    address token;
    uint256 tokenId;
    address drawingToken;
    uint256 drawingTokenStartId;
    uint256 drawingTokenEndId;

    /// @audit pack all of this into one slot
    uint96 drawBufferTime;
    uint96 recoverTimelock;
    uint64 subscriptionId;

    bytes32 keyHash; 
}
```

- For `CurrentRequest`:

```solidity=
struct CurrentRequest {
    uint256 currentChainlinkRequestId;
    uint256 currentChosenTokenId;
    bool hasChosenRandomNumber; /// @audit one slot
    uint256 drawTimelock; /// @audit one slot
}
```
Change to
```solidity=
struct CurrentRequest {
    uint256 currentChainlinkRequestId;
    uint256 currentChosenTokenId;
    bool hasChosenRandomNumber;
    uint248 drawTimelock; /// @audit reduce this to fit in the slot above
}
```

We save a total of 3 storage slots every time `VRFNFTRandomDraw` is created from the factory, which equates to saving both factory-deployment gas and storage-access gas, due to having to work with fewer slots. Totals to about **63000** gas (21000 gas per Gsset saved), and potentially many more during operations.

## [G-02] Check all parameters before writing into storage

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L80

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L215-L221

It is recommended to write into storage only after all parameters are validated, and any reverting tests are passed. Otherwise, if the function reverts, massive amount of gas will be wasted on a storage write.

Saves many Gsset instances (costing **21000** gas per storage slot, for a maximum of **~189000** gas if all fields are non-zero).

## [G-03] Cache storage variables into memory to avoid re-reading

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L277-L300

```javascript
function winnerClaimNFT() external { // @audit `settings` is accessed many times from storage
```

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L141-L169

```javascript
function _requestRoll() internal { // @audit `request` is accessed many times from storage
```

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L304-L320

```javascript
emit OwnerReclaimedNFT(owner()); // @audit owner is re-accessed later
```

Given that the relevant storage are accessed many times throughout the function and not fully modified, it is recommended to cache any storage reads into memory, and only read from there.

Converts any Gwarmaccess (**100 gas**) into much cheaper stack read (**3 gas**). The structs takes up many storage slots, so the impact is actually higher depending on application.

## [G-04] Perform all operations before emitting event

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L123

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L180

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L287-L292

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L312

All of the following events perform some kind of storage access. However should any of the activities coming after it reverts (e.g. condition checks, NFT transfer), the storage access will have been redundant. In some cases this equates to a wasted *Gwarmaccess*, costing **2100 gas per storage slot**

It is recommended to emit the event only after all operations have been completed. In the maximum case this can save **18900** gas.

## [G-05] `startDraw`: Redundant condition check

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L175-L177

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L204-L206

Both of the condition checks are redundant, for an exact same check is performed later on. It is no need to check the same condition twice.

Saves one *Gwarmaccess* (**100 gas**) on reducing one storage read per check.

## [G-06] Repeated information in emitted event

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L287-L292

Several information in the `settings` struct are emitted multiple times, which comes with redundant info accesses. It is recommended to only emit the entire `settings` struct, similar to other events, saving two Gwarmaccess (**100 gas each**).

## [G-07] `redraw`: It is possible to trick `_requestRoll` into rerolling with much less gas cost

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L209-L212

The operation `delete` will simply set all variables to its default value that is zero. Given that the `_requestRoll` function right after it will set all fields in `request` to a non-zero value anyway, the total gas cost of the operation will be
- Four Gsreset (2900 gas each) for setting storage to zero
- Four Gsset (21000 gas each) for setting storage from zero to non-zero

We can save two Gsset by only setting `request.currentChainlinkRequestId` and `request.hasChosenRandomNumber` to zero before calling the function. All of the checks will pass, and function will have worked as normal. Saves **42000** gas in total.

## [G-08] `InitializedDraw`: Don't use storage access for event

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L123

Given that `_settings` is already a function parameter, and is not modified, accessing the memory data is much cheapter than accessing storage (see [G-03]).

This finding is similar to [G-04], except is applicable even for non-reverting operations.

Saves **97** gas per slot, for **873** gas in total.

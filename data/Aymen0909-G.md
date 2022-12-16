# Gas Optimizations

## Summary

|               | Issue         | Instances     |
| :-------------: |:-------------|:-------------:|
| 1     | Variables inside `struct` should be packed to save gas  | 2  |
| 2     | Use `unchecked` blocks to save gas  |  2 |
| 3     | `storage` variable should be cached into `memory` variables instead of re-reading them  |  4 |
| 4     | `memory` values should be emitted in events instead of `storage` ones  |  2 |


## Findings

### 1- Variables inside `struct` should be packed to save gas :

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

There are 2 instances of this issue:

File: src/interfaces/IVRFNFTRandomDraw.sol [Line 59-68](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L59-L68)

```
struct CurrentRequest {
    /// @notice current chainlink request id
    uint256 currentChainlinkRequestId;
    /// @notice current chosen random number
    uint256 currentChosenTokenId;
    /// @notice has chosen a random number (in case random number = 0(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0))
    bool hasChosenRandomNumber;
    /// @notice time lock (block.timestamp) that a re-draw can be issued
    uint256 drawTimelock;
}
```

As the `drawTimelock` variable represents time unit (in seconds) it can't really overflow 2^128 (wich give 1.0783e+31 years when converted), and the `currentChosenTokenId` value is always ranged between `drawingTokenStartId` and `drawingTokenEndId` (bcause those two values represent real NFT token IDs they can't overflow in reality) so it can't really overflow 2^128, those variables values should be packed together and the struct should be modified as follow to save gas :

```
struct CurrentRequest {
    /// @notice current chainlink request id
    uint256 currentChainlinkRequestId;
    /// @notice time lock (block.timestamp) that a re-draw can be issued
    uint128 drawTimelock;
    /// @notice current chosen random number
    uint128 currentChosenTokenId;
    /// @notice has chosen a random number (in case random number = 0(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0))
    bool hasChosenRandomNumber;
}
```


File: src/interfaces/IVRFNFTRandomDraw.sol [Line 71-90](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L71-L90)

```
struct Settings {
    /// @notice Token Contract to put up for raffle
    address token;
    /// @notice Token ID to put up for raffle
    uint256 tokenId;
    /// @notice Token that each (sequential) ID has a entry in the raffle.
    address drawingToken;
    /// @notice Start token ID for the drawing (if totalSupply = 20 but the first token is 5 (5-25), setting this to 5 would fix the ordering)
    uint256 drawingTokenStartId;
    /// @notice End token ID for the drawing (exclusive) (token ids 0 - 9 would be 10 in this field)
    uint256 drawingTokenEndId;
    /// @notice Draw buffer time – time until a re-drawing can occur if the selected user cannot or does not claim the NFT.
    uint256 drawBufferTime;
    /// @notice block.timestamp that the admin can recover the NFT (as a safety fallback)
    uint256 recoverTimelock;
    /// @notice Chainlink gas keyhash
    bytes32 keyHash;
    /// @notice Chainlink subscription id
    uint64 subscriptionId;
}
```

Here we have the following :

The variabales `tokenId`, `drawingTokenStartId` and `drawingTokenEndId` represent real NFT collection ids so they can't realy overflow 2^64 as most NFT collection have a max supply and even if they don't the token id can't realistically be greater than 2^64 = 1.8446744e+19 

The variabales `drawBufferTime` and `recoverTimelock` represent time unit and they have both upper and lower bond :

`HOUR_IN_SECONDS <= drawBufferTime <= MONTH_IN_SECONDS`

`time + WEEK_IN_SECONDS <= recoverTimelock <= time + 1 year`

Thus those two values can't realistically overflow 2^64 (which is equivalent to 584554046918.09 year)

So all the previously mentionned variabales should be packed together to save gas and the struct should be modified as follow :

```
struct Settings {
    /// @notice Token Contract to put up for raffle
    address token;
    /// @notice Token that each (sequential) ID has a entry in the raffle.
    address drawingToken;
    /// @notice Token ID to put up for raffle
    uint64 tokenId;
    /// @notice Start token ID for the drawing (if totalSupply = 20 but the first token is 5 (5-25), setting this to 5 would fix the ordering)
    uint64 drawingTokenStartId;
    /// @notice End token ID for the drawing (exclusive) (token ids 0 - 9 would be 10 in this field)
    uint64 drawingTokenEndId;
    /// @notice Draw buffer time – time until a re-drawing can occur if the selected user cannot or does not claim the NFT.
    uint64 drawBufferTime;
    /// @notice block.timestamp that the admin can recover the NFT (as a safety fallback)
    uint64 recoverTimelock;
    /// @notice Chainlink subscription id
    uint64 subscriptionId;
    /// @notice Chainlink gas keyhash
    bytes32 keyHash;
}
```

### 2- Use `unchecked` blocks to save gas :

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

There are 2 instances of this issue:

File: src/VRFNFTRandomDraw.sol [Line 159](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L159)
```
request.drawTimelock = block.timestamp + settings.drawBufferTime; 
```

The above operation cannot overflow because `request.drawTimelock` represent a time measurement (in seconds) and we are adding to the current timestamp a bounded value `settings.drawBufferTime` as illustated [here](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L83-L88) thus it can't realistically overflow so the operation should be marked unchecked. 


File: src/VRFNFTRandomDraw.sol [Line 249](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L249-L250)
```
uint256 tokenRange = settings.drawingTokenEndId - settings.drawingTokenStartId;
```

The above operation cannot underflow because of the check : 

File: src/VRFNFTRandomDraw.sol [Line 112](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L112-L117)
```
if (
    _settings.drawingTokenEndId < _settings.drawingTokenStartId ||
    _settings.drawingTokenEndId - _settings.drawingTokenStartId < 2
) {
    revert DRAWING_TOKEN_RANGE_INVALID();
}
```

So the operation should be marked unchecked.


### 3- `storage` variables should be cached into `memory` variables instead of re-reading them from `storage` :

The instances below point to the second+ access of a state variable within a function, the caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read, thus saves **100gas** for each instance.

There are 4 instances of this issue :

File: src/VRFNFTRandomDraw.sol [Line 149-154](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L149-L154)

```
if (
    request.hasChosenRandomNumber &&
    // Draw timelock not yet used
    request.drawTimelock != 0 &&
    request.drawTimelock > block.timestamp
)
```

The value of `request.drawTimelock` is read twice from storage, the above code should be modified as follow to save gas :
```
uint256 drawTimeLock = request.drawTimelock;
if (
    request.hasChosenRandomNumber &&
    // Draw timelock not yet used
    drawTimeLock != 0 &&
    drawTimeLock > block.timestamp
)   
```

File: src/VRFNFTRandomDraw.sol [Line 249-256](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L249-L256)

```
// Get total token range
uint256 tokenRange = settings.drawingTokenEndId -
    settings.drawingTokenStartId;

// Store a number from it here (reduce number here to reduce gas usage)
// We know there will only be 1 word sent at this point.
request.currentChosenTokenId =
    (_randomWords[0] % tokenRange) +
    settings.drawingTokenStartId;
```

The value of `settings.drawingTokenStartId` is read twice from storage, the above code should be modified as follow to save gas :
```
uint256 startId = settings.drawingTokenStartId;

// Get total token range
uint256 tokenRange = settings.drawingTokenEndId - startId;

// Store a number from it here (reduce number here to reduce gas usage)
// We know there will only be 1 word sent at this point.
request.currentChosenTokenId = (_randomWords[0] % tokenRange) + startId;    
```

File: src/VRFNFTRandomDraw.sol [Line 287-299](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L287-L299)

```
emit WinnerSentNFT(
    user,
    address(settings.token),
    settings.tokenId,
    settings
);

// Transfer token to the winter.
IERC721EnumerableUpgradeable(settings.token).transferFrom(
    address(this),
    msg.sender,
    settings.tokenId
);   
```

The values of `settings.token` and `settings.tokenId` are read twice from storage, the above code should be modified as follow to save gas :
```
address token = settings.token;
uint256 tokenId = settings.tokenId;

emit WinnerSentNFT(
    user,
    token,
    tokenId,
    settings
);

// Transfer token to the winter.
IERC721EnumerableUpgradeable(token).transferFrom(
    address(this),
    msg.sender,
    tokenId
);     
```

File: src/VRFNFTRandomDraw.sol [Line 312-319](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L312-L319)

```
// Send event for indexing that the owner reclaimed the NFT
emit OwnerReclaimedNFT(owner());

// Transfer token to the admin/owner.
IERC721EnumerableUpgradeable(settings.token).transferFrom(
    address(this),
    owner(),
    settings.tokenId
); 
```

The `owner()` method is called twice and it will read the owner address twice from storage which costs more gas, and because here the `lastResortTimelockOwnerClaimNFT()` function can only be called by the owner so we will always have `msg.sender == owner()`, the above code should be modified as follow to save gas :
```
address owner = msg.sender;

// Send event for indexing that the owner reclaimed the NFT
emit OwnerReclaimedNFT(owner);

// Transfer token to the admin/owner.
IERC721EnumerableUpgradeable(settings.token).transferFrom(
    address(this),
    owner,
    settings.tokenId
);     
```

### 4- `memory` values should be emitted in events instead of `storage` ones :

Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead, this will save **~101 GAS**

There are 2 instances of this issue :

File: src/VRFNFTRandomDraw.sol [Line 123](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L123)
```
// Emit initialized event for indexing
emit InitializedDraw(msg.sender, settings);
```

The storage `settings` variable is emitted which cost more gas, should instead emit memory variables `_settings` as follow :
```
// Emit initialized event for indexing
emit InitializedDraw(msg.sender, _settings);
```

File: src/VRFNFTRandomDraw.sol [Line 312](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L312)
```
// Send event for indexing that the owner reclaimed the NFT
emit OwnerReclaimedNFT(owner());
```

The `owner()` method read from storage the value of owner address but as the `lastResortTimelockOwnerClaimNFT()` function can only be called by the owner so we will always have `msg.sender == owner()`, and thus the `msg.sender` value should be emitted to save gas :
```
// Send event for indexing that the owner reclaimed the NFT
emit OwnerReclaimedNFT(msg.sender);
```
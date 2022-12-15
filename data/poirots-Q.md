# QA Report

## Low Risk Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [L-01] | `redraw()` should be called by anyone | 1 |
| [L-02] | `IERC721EnumerableUpgradeable` may lead to false assumptions | 6 |
| [L-03] | `drawingTokenEndId` should be inclusive or altered to a range | 1 |
| [L-04] | An `owner` can resign and lead to locked NFT | 1 |
| [L-05] | `fulfillRandomWords` must not revert | 1 |

Total 5 issues

## Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [N-01] | `getRequestDetails()` should include the tokenid |1|
| [N-02] | Avoid setting time variables manually  |1|
| [N-03] | Use constants instead of immutable variables | 1 |
| [N-04] | Uppercase immutable variables | 6 |
| [N-05] | Empty blocks should be avoided | 1 |
| [N-06] | Missing _NatSpec_ | 1 |
| [N-07] | Contracts that extend interfaces should override its methods | 3 |
| [N-08] | `_requestRoll()` after confirming that the raffle is viable  |1|

Total 8 issues

## [L-01] `redraw()` should be called by anyone

[VRFNFTRandomDraw.sol#L203-L225](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L203-L225) 

Redrawing a raffle is already protects the winner through the timelocking mechanism. Dependency on the owner should be avoidable in this instance by removing the modifier `onlyOwner`, allowing anyone to redraw the raffle.

## [L-02]  `IERC721EnumerableUpgradeable` may lead to false assumptions

Throughout the [contract](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol), there's a wrapper of NFT collections to `IERC721EnumerableUpgradeable` instances:

```bash
src/VRFNFTRandomDraw.sol:127:            IERC721EnumerableUpgradeable(_settings.token).ownerOf(
src/VRFNFTRandomDraw.sol:187:            IERC721EnumerableUpgradeable(settings.token).transferFrom(
src/VRFNFTRandomDraw.sol:216:            IERC721EnumerableUpgradeable(settings.token).ownerOf(
src/VRFNFTRandomDraw.sol:271:            IERC721EnumerableUpgradeable(settings.drawingToken).ownerOf(
src/VRFNFTRandomDraw.sol:295:        IERC721EnumerableUpgradeable(settings.token).transferFrom(
src/VRFNFTRandomDraw.sol:315:        IERC721EnumerableUpgradeable(settings.token).transferFrom(
```

This could lead to false assumptions when working with this contract (particularly when considering how settings are defined).

Consider altering to `IERC721` if the goal is to allow any NFT collection compliant with EIP-721.

## [L-03] `drawingTokenEndId` should be inclusive or altered to a range

[Natspec](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L80-L81) specifies that the last ID is exclusive in the raffle, but the variable's name could lead to wrong assumptions. 

Consider altering the logic to the contract to include the ID or to change the logic to a range definition, since it is only used twice ([1](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L249),[2](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L113-L114)) and could avoid misinterpretations.

## [L-04] An owner can resign and lead to locked NFTs

Since there's a possibility of unclaimed drafts, the owner is the only one able to rescue the prize NFT from the raffle contract. Thus, having the ability to resign ownership (including non-intentional) could lead to stuck NFTs.

Consider altering or removing [resignOwnership](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L112-L116) method: 

```solidity
    /// @notice Resign ownership of contract
    /// @dev only callably by the owner, dangerous call.
    function resignOwnership() public onlyOwner {
        _transferOwnership(address(0));
    }
```

## [L-05] `fulfillRandomWords` must not revert

Accordingly to ChainLinks' [documentation]():

> `fulfillRandomWords` must not revert
> If your `fulfillRandomWords()` implementation reverts, the VRF service will not attempt to call it a second time. Make sure your contract logic does not revert. Consider simply storing the randomness and taking more complex follow-on actions in separate contract calls made by you, your users, or an Automation Node.

This project current implementation does revert [in two instances](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L230-L260), although they are not expected to materialize.

Nevertheless, consider altering the logic to drop the random generated whenever the requestId does not match and ignore extra words if the array received is greater than the expected amount.

## [N-01] `getRequestDetails()` should include the tokenid

In [VRFNFTRandomDraw.sol#getRequestDetails()](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L58-L70) should include `currentChosenTokenId` ([at](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L63)) and ease integrations with other tools.


## [N-02] Avoid setting time variables manually

Use solidity [Time Units](https://docs.soliditylang.org/en/v0.8.17/units-and-global-variables.html#time-units) to avoid mistakes in defining time variables. In [VRFNFTRandomDraw.sol#L28-L33](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L28-L33) (the `MONTH_IN_SECONDS` leads to a medium issue): 

```solidity

/// @dev 60 seconds in a min, 60 mins in an hour
uint256 immutable HOUR_IN_SECONDS = 60 * 60;
/// @dev 24 hours in a day 7 days in a week
uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
// @dev about 30 days in a month
uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;

```

Consider changing to:

```solidity

/// @dev 60 seconds in a min, 60 mins in an hour
uint256 immutable HOUR_IN_SECONDS = 1 hours;
/// @dev 24 hours in a day 7 days in a week
uint256 immutable WEEK_IN_SECONDS = 1 weeks;
// @dev about 30 days in a month
uint256 immutable MONTH_IN_SECONDS = 30 days;

```

## [N-03] Use constants instead of immutable variables

Variables defined in [VRFNFTRandomDraw.sol#L21-L33](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L21-L33) should be constants, since they aren't define at contract creation:

```solidity
    uint32 immutable callbackGasLimit = 200_000;
    /// @notice Chainlink request confirmations, left at the default
    uint16 immutable minimumRequestConfirmations = 3;
    /// @notice Number of words requested in a drawing
    uint16 immutable wordsRequested = 1;

    /// @dev 60 seconds in a min, 60 mins in an hour
    uint256 immutable HOUR_IN_SECONDS = 60 * 60;
    /// @dev 24 hours in a day 7 days in a week
    uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
    // @dev about 30 days in a month
    uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```

If these are rules, consider changing them to `IVRFNFTRandomDraw` interface:

```solidity
    uint32 constant callbackGasLimit = 200_000;
    /// @notice Chainlink request confirmations, left at the default
    uint16 constant minimumRequestConfirmations = 3;
    /// @notice Number of words requested in a drawing
    uint16 constant wordsRequested = 1;

    /// @dev 60 seconds in a min, 60 mins in an hour
    uint256 constant HOUR_IN_SECONDS = 60 * 60;
    /// @dev 24 hours in a day 7 days in a week
    uint256 constant WEEK_IN_SECONDS = (3600 * 24 * 7);
    // @dev about 30 days in a month
    uint256 constant MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```

## [N-04] Uppercase immutable variables

In [VRFNFTRandomDraw.sol#L22-L26](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L22-L26):

```solidity
    uint32 immutable callbackGasLimit = 200_000;
    /// @notice Chainlink request confirmations, left at the default
    uint16 immutable minimumRequestConfirmations = 3;
    /// @notice Number of words requested in a drawing
    uint16 immutable wordsRequested = 1;
```

In [VRFNFTRandomDraw.sol#L37](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L37):

```solidity
    VRFCoordinatorV2Interface immutable coordinator;
```

In [Version.sol#L5](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol#L5):
```solidity
  uint32 private immutable __version;
```

In [VRFNFTRandomDrawFactory.sol#L21](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L21):
```solidity
    address public immutable implementation;
```


## [N-05] Empty blocks should be avoided

Avoid using code blocks, such as:

In [VRFNFTRandomDrawFactory.sol#L53-L59](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L53-L59):

```solidity
    /// @notice Allows only the owner to upgrade the contract
    /// @param newImplementation proposed new upgrade implementation
    function _authorizeUpgrade(address newImplementation)
        internal
        override
        onlyOwner
    {}
```

Consider emiting an event.


## [N-06] Missing _NatSpec_

Consider adding specification to the following code blocks:

In [IVRFNFTRandomDraw.sol#L28](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L28):
```solidity
    error REDRAW_TIMELOCK_NEEDS_TO_BE_LESS_THAN_A_MONTH();
```

## [N-07] Contracts that extend interfaces should override its methods

Considering using the `override` keyword to indicate  which methods are implementing the interface.

For `VRFNFTRandomDraw` regarding `IVRFNFTRandomDraw`: `initialize`, `startDraw`, `redraw`, `hasUserWon`, `winnerClaimNFT`, `lastResortTimelockOwnerClaimNFT`, `getRequestDetails`.
For `VRFNFTRandomDrawFactory` regarding `IVRFNFTRandomDrawFactory`: `initialize`, `startDraw`.


## [N-08] `_requestRoll()` after confirming that the raffle is viable 

In [startDraw()](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L173-L198), the contract makes a request for a random number before confirming that it has the prize to raffle. 

Consider confirming first that the contract has the NFT to raffle before wasting resources calling for a random.
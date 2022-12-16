
# Low Severity findings

## drawingTokenEndId will not be chosen

In function `fulfillRandomWords` winning tokenID is chosen by adding the starting tokenId with the result of the modulo operation. Since the result of modulo operation will be between 0 and tokenRange-1, last tokenId will never be chosen as the winner

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L255

```
        request.currentChosenTokenId =
            (_randomWords[0] % tokenRange) +
            settings.drawingTokenStartId;
```

### Recommended Mitigation

Increment the tokenRange so that result be between 0 and tokenRange

```        uint256 tokenRange = settings.drawingTokenEndId -
            settings.drawingTokenStartId + 1;
```

## OwnableUpgradeable does not have storage gap

OwnableUpgradeable contract does not contain storage gaps which will cause errors in future upgrades involving new variables

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L14

```
    contract VRFNFTRandomDrawFactory is
        IVRFNFTRandomDrawFactory,
        OwnableUpgradeable,
        UUPSUpgradeable,
        Version(1)
    {
```

### Recommended Mitigation

Add storage gap variable to the contract

## `_requestRoll` does not revert when `drawTimelock == block.timestamp`

In function `redraw` redraw reverts when drawTimelock is equal to block.timestamp, but in `_requestRoll` function executes even when drawTimelock is equal to blocktimestamp and the value is updated. So users will be unable to claim NFT at drawTimelock.

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L204

```
        if (request.drawTimelock >= block.timestamp) {
            revert TOO_SOON_TO_REDRAW();
        }
```

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L149

```
        if (
            request.hasChosenRandomNumber &&
            // Draw timelock not yet used
            request.drawTimelock != 0 &&
            request.drawTimelock > block.timestamp  
        ) {
            revert STILL_IN_WAITING_PERIOD_BEFORE_REDRAWING();
        }
```

### Recommended Mitigation

Change condition in `_requestRoll`

```
        if (
            request.hasChosenRandomNumber &&
            // Draw timelock not yet used
            request.drawTimelock != 0 &&
            request.drawTimelock >= block.timestamp  
        ) {
            revert STILL_IN_WAITING_PERIOD_BEFORE_REDRAWING();
        }
```

# Non critical findings

## Owner can withdraw NFT during draw

In function `lastResortTimelockOwnerClaimNFT`, owner can withdraw NFT after recoverTimelock, but it does not check whether the contract is in draw state. When owner withdraws NFT after recoverTimelock but before drawTimelock ends, a user with a winning tokenId will be unable to claim the NFT.

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L306

```
    function lastResortTimelockOwnerClaimNFT() external onlyOwner { 
        // If recoverTimelock is not setup, or if not yet occurred
        if (settings.recoverTimelock > block.timestamp) {
            // Stop the withdraw
            revert RECOVERY_IS_NOT_YET_POSSIBLE();
        }

        // Send event for indexing that the owner reclaimed the NFT
        emit OwnerReclaimedNFT(owner()); 

        // Transfer token to the admin/owner.
        IERC721EnumerableUpgradeable(settings.token).transferFrom(
            address(this),
            owner(),   
            settings.tokenId
        );
    }
```

### Recommended Mitigation

Add check to prevent NFT withdraw during draw

```
        if (settings.recoverTimelock > block.timestamp || request.drawTimelock >= block.timestamp) {
            // Stop the withdraw
            revert RECOVERY_IS_NOT_YET_POSSIBLE();
        }
```


## resignOwnership will cause NFT to be stuck in the contract

If the owner resigns ownership before re-draw, NFT cannot be withdrawn and will be stuck in the contract

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/ownable/OwnableUpgradeable.sol#L114

```
    function resignOwnership() public onlyOwner {
        _transferOwnership(address(0));
    }
```
### Recommended Mitigation

Prevent resigning when NFT is in the contract, remove the function

## Use native time units instead of numbers 

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L29

```
    uint256 immutable HOUR_IN_SECONDS = 60 * 60;
    /// @dev 24 hours in a day 7 days in a week
    uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
    // @dev about 30 days in a month
    uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```

### Recommended Mitigation

Use time units `hours`, `weeks`

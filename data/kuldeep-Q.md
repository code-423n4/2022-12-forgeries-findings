# Low/Non-critical 1
`if conditions` in the `_requestRoll` method will always be false. They never revert in any situation.

## Explanation

The `_requestRoll` internal method is called only by 2 methods i.e. [startDraw](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L183) and [redraw](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L212).

The `_requestRoll` has [2 `if conditions`](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L144). Let us look at that one by one.

First condition:

`if (request.currentChainlinkRequestId != 0) {
            revert REQUEST_IN_FLIGHT();
 }`

Second condition:

`if (
            request.hasChosenRandomNumber &&
            request.drawTimelock != 0 &&
            request.drawTimelock > block.timestamp
        ) {
            revert STILL_IN_WAITING_PERIOD_BEFORE_REDRAWING();
 }`

Let us examine:

### startDraw calls _requestRoll

1. When `startDraw` will call `_requestRoll` and the first condition is checked, `request.currentChainlinkRequestId` will be 0 only hence `request.currentChainlinkRequestId` will be set in this call. Now `startDraw` method will never be able to call the `_requestRoll` method as this itself has the same check which will always hold true.

2. Similarly for the 2nd condition, when it will be checked the first time, hasChosenRandomNumber will be false. Now `startDraw` method will never be able to call the `_requestRoll` method as this itself has another check which will always hold.

### redraw calls _requestRoll

1. When `redraw` calls the` _requestRoll` method, it first [deleted the `request` state](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L209) which will initialize every variable in the struct with the default value. After this, the conditions in `requestRoll` will always be false.


## Mitigation
Remove both conditions on the `_requestRoll` method at least for now as they serve no purpose. When all tests are run after removing these conditions, still all test cases are successful with some less gas usage.


# Low/Non-critical 2
As per the requirement, recoverTimeLock needs to be at least 1 week. Although logic forces it to be more than 1 week always.

## Explanation
This condition check [here](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L90) will revert if the `recoverTimelock` value is 1 week exactly due to `less than` in the check.

## Mitigation
The condition should be updated as `_settings.recoverTimelock <= block.timestamp + WEEK_IN_SECONDS` to allow users to set recoverTimelock exactly 1 week.

# Informational 1
Typo [here](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L294). It should be `winter` instead of `winner`.

# Informational 2
Wrong comment [here](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L305).
`settings.recoverTimelock` will always be set up and greater than 1 week because of this [check](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L90) in `initialize` method.

Updated comment as: // If recoverTimelock is not yet occurred

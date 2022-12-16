## Unnecessary check in `VRFNFTRandomDraw`
`_requestRoll` function in `VRFNFTRandomDraw` includes an unnecessary check. It requires that `request.currentChainlinkRequestId == 0`. 
```
        if (request.currentChainlinkRequestId != 0) {
            revert REQUEST_IN_FLIGHT();
        }
```
However, there is never a case where the function would be called and the requirement would not be met.
`_requestRoll` is an internal function an it is called only in two cases:
1. `startDraw` function, which includes the same check.
2. `redraw` function, which deletes the `request` before executing `_requestRoll`, meaning `request.currentChainlinkRequestId` will be `0`.

### Recommendation
Remove the lines 144 to 146 in `VRFNFTRandomDraw.sol` (https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L144-L146).

## Largest uint256 can never win
There are two variables that define the range of tokens that participate in a raffle. The `settings.drawingTokenStartId` and `settings.drawingTokenEndId`. If the first one is 5 and last one is 10, that includes only tokens with ID of 5 to 9. The `settings.drawingTokenEndId` is excluded. This means that if there is a token with ID equal to the largest `uint256` number, it will always be excluded from a raffle. Although it is uncommon for a token to have that high ID number, it is not impossible. There is no rules for how token IDs should be counted, therefore someone can decide to create an NFT collection, where IDs start at the last `uint256` number.

### Recommendation
Although the above scenario might be unlikely, it would be recommended to make the `settings.drawingTokenEndId` inclusive. To do that you only need to add `+ 1` to the calculation of `tokenRange` in [`fulfillRandomWords`](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L230) function.
```
uint256 tokenRange = settings.drawingTokenEndId - settings.drawingTokenStartId + 1;
```

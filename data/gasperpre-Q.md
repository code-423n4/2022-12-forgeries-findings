## Finding 1
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
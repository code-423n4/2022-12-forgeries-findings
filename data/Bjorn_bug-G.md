#### Unnecessary duplicate sanity checks in _requestRoll() & startDraw()
### Affected line of code:
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L144
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L175

**if (request.currentChainlinkRequestId != 0) { revert REQUEST_IN_FLIGHT();}** will be the first condition to be reverted if the condition does not meet when calling the function **startDraw()** without even arriving at calling the internal function **_requestRoll()** who was programmed to re-verify the same condition again (in case the first one passed).
 
### Impact:
Waste of gas.

### Recommended mitigation steps:
Consider removing one of the sanity checks **if (request.currentChainlinkRequestId != 0) {revert REQUEST_IN_FLIGHT();}** in one of the above functions

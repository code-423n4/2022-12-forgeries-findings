## Impact
In determining if the contest can be re-drawn, the `redraw ()` function checks the `request.drawTimelock` is greater than or equals  to `block.timestamp` as seen here https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L204. 
However, the `_requestRoll ()` function which is used in starting a re-draw and/or re-drawing also performs a similar check but only checks if `request.drawTimelock > block.timestamp` as seen here: https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L153

It would be better that a consistent condition be determined in checking the redraw period
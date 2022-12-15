https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L33
- This is actually 30 weeks, not 30 days. Validation related to `MONTH_IN_SECONDS` will be affected.

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L142-L156
- In the current code, `_requestRoll()` will never be called in cases where any if these if conditionals will be true; the reverts cannot be triggered

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L161
- Incorrect comment, this is not necessarily the first time `requestRandomWords()` is called

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L203
- Recommend that `redraw()` emit an event like `startDraw()` does so users will be aware the raffle is being re-run.

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L294
- typo in comment

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L295
- recommend using IERC721 `safeTransferFrom()`

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L315
- recommend using IERC721 `safeTransferFrom()`


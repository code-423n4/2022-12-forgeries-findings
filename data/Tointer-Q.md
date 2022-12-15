1) https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L306
`recoverTimelock` is set in `initialize` function, but actual draw starting when `startDraw` is called. Therefore, owner can wait for `lastResortTimelockOwnerClaimNFT` function to become available, then call `startDraw`, and if result would not be satisfactory, call `lastResortTimelockOwnerClaimNFT` and get their NFT back.

    Solution: start countdown to last resort claim in `startDraw` function

2) https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L127 https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L187 https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L216
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L271 
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L295 
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L315
This casts to `IERC721EnumerableUpgradeable` is confusing, since we are not actually call any Enumerable logic (and if we tried, we would fail for many collections, including BAYC). It's better to cast to IERC721

3) https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L142-L156
Redundant code. `_requestRoll()` is called either right after `delete request`, or in `startDraw` function when `request` variable are not initialized. Therefore `request.hasChosenRandomNumber` would always be false and `request.currentChainlinkRequestId` would always be 0.
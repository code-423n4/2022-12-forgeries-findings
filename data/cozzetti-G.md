### Redundant check in `startDraw`

The [initial check](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L175-L177) in the `startDraw` function of the `VRFNFTRandomDraw` contract is redundant, because the condition is already checked by [the internal call](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L183) to `_requestRoll`. Remove the check from `startDraw` to improve gas efficiency.

### Redundant try/catch in `startDraw`

The `transferFrom` function of an ERC721 token would already revert if a token ID hasn't been approved. Therefore, [the `try/catch` structure used in `startDraw`](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L186-L194) is redundant and can be removed to save gas.
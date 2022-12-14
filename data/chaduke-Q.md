GA1. https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/utils/Version.sol#L2
It is advised that all contracts should be locked with the newest version of solidity 0.8.17.

GA2. https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L2
Lock all contracts with the most current version of Solidity 0.8.17


GA3: 
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L47
Zero address check for ``coordinator`` to avoid redeployment of the contract. 

GA4: https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L80
Move this line (effects) to the last line of the ``initialize`` function can save gas. 

GA5: https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L65
Using 1 to encode false and 2 to encode true will saving gas since changing from zero to non-zero is more expensive

GA6. https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L126-L137
Simplifying the try-catch stmt can save gas see below:
```
if(IERC721EnumerableUpgradeable(_settings.token).ownerOf(_settings.tokenId) != admin) revert DOES_NOT_OWN_NFT();
```
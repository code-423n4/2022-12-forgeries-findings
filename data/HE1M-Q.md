## No. 1
During initializing a draw, the protocol does not check that the token id of NFT collection (drawing token) exists or not. For example, the owner uses crypto punk as drawing token and sets `drawingTokenStartId = 0` and `drawingTokenEndId = 10100`, while the total supply of crypto punk is **10000**. So, if, after the draw, the chosen token id is equal to 10050, no one can win the NFT, because such token id does not exists in crypto punk collection. So, it is better to have the following check in the function `initialize` to prevent such mistakes:
```
require(_settings.drawingTokenEndId <  IERC721EnumerableUpgradeable(settings.drawingToken).totalSupply());
``` 
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L75

## No. 2
Deleting the `request` is not done consistently. It is only done during redrawing:
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L209
But, it is better to delete the `request` also when:
 - the owner is claiming his own NFT.
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L304
 - the winner is claiming the NFT.
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L277
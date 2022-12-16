Events are emitted before ERC721 transferFrom() is performed. In some situations transferFrom fails and the result is not checked by default. But the front-end becomes aware that the transfer is successful. But the actual transfer has not been successful on chain.


https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L287
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L312
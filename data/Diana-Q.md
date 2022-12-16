
## 01 USING TRANSFERFROM FUNCTION OF AN ERC721 CONTRACT MAY FREEZE THE USER'S NFT

When using the `transferFrom` function of an ERC721 contract to send an NFT, if the receiving address is a smart contract and does not support ERC721, the NFT can be frozen in the contract.

_There are 3 instances of this issue_

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol

```
File: src/VRFNFTRandomDraw.sol

187: IERC721EnumerableUpgradeable(settings.token).transferFrom(
295: IERC721EnumerableUpgradeable(settings.token).transferFrom(
315: IERC721EnumerableUpgradeable(settings.token).transferFrom(
```

### Recommended Mitigation Steps

Use the ERC721 contract’s `safeTransferFrom` function to send NFTs.

--------------------

## 02 USE A MORE RECENT VERSION OF SOLIDITY

It's a best practice to use the latest compiler version.

The specified minimum compiler version is quite old. Older compilers might be susceptible to some bugs. We recommend changing the solidity version pragma to the latest version to enforce the use of an up to date compiler.

List of known compiler bugs and their severity can be found here: [https://etherscan.io/solcbuginfo](https://etherscan.io/solcbuginfo)

_This issue exists in all the In-scope contracts_

--------

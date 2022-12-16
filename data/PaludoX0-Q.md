## Make VRFNFTRandomDrawFactory.sol pausable
It is suggested to make the the contract pausable or at least makeNewDraw function, it could be helpful in case something goes wrong or you have to upgrade the contract.
You can inherit openzeppelin pausable contract https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/security/Pausable.sol



## Typo in comment
**VRFNFTRandomDraw.sol#L294**
Comment "winter" should be "winner"



## Sanity check if drawingTokenEndId exist
**VRFNFTRandomDraw.sol#L118**
It is possible to verify that token id which correpond to drawingTokenEndId exists: 
`if( drawingTokenEndId > (drawingToken.length) {revert}`

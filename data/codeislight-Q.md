- VRFNFTRandomDraw.initialize() lacks checking the existential of drawingTokenStartId and drawingTokenEndId, it is suggested that a staticCall to be made and check if it successed within a try catch block and check the returned value is not equal to zero address in the case of irregular contracts that don't revert on non-existential tokenIds.
- VRFNFTRandomDrawFactory.constructor() needs to ensure code size of _implementation is greater than zero, since it's possible for a non zero address to be an EOA.
- VRFNFTRandomDraw.startDraw() there is no need to check that currentChainlinkRequestId must be equal to zero, since it's alredy checked in _requestRoll internal function.
- VRFNFTRandomDraw.initialize() must check keyHash to be one of the corresponding keyHashes for GasPrice supported by the coordinator:
https://vrf.chain.link/mainnet
 Note that chainlink coordinator doesn't check the validity of keyHashes to save on gas fees on their end, refer to:
https://github.com/smartcontractkit/chainlink/blob/e1e78865d4f3e609e7977777d7fb0604913b63ed/contracts/src/v0.8/VRFCoordinatorV2.sol#L407
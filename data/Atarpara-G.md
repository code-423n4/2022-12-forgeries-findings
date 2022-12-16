
#### G-01 Setting struct can be check in factory

`VRFNFTRandomDraw.sol#initialize()` checking for `setting` parameter but it can be check into `VRFNFTRandomDrawFactory.sol#makeNewDraw` if there any invalid value it will revert there without deploying proxy contract and user will save proxy deployment and initialize() external call cost
(around 34k gas will save).



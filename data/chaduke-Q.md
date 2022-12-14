GA1. https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/utils/Version.sol#L2
It is advised that all contracts should be locked with the newest version of solidity 0.8.17.

GA2. https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L2
Lock all contracts with the most current version of Solidity 0.8.17


GA3: 
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L47
Zero address check for ``coordinator`` to avoid redeployment of the contract. 

GA4: https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L47-L52
No need to introduce the state variable ``coordinator`` since the parent contract   ``VRFConsumerBaseV2`` already has ``vrfCoordinator`` for the same purpose. 



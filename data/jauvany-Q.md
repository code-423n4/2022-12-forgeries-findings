Different pragma solidity

Vulnerability details

Version.sol has a different pragma statement than the rest, it uses version ^0.8.10 .

It's cleaner to use the same solidity versions.

Proof of Concept

Version.sol: pragma solidity ^0.8.10;
VRFNFTRandomDrawFactoryProxy.sol: pragma solidity 0.8.16;
VRFNFTRandomDrawFactory.sol: pragma solidity 0.8.16;
VRFNFTRandomDraw.sol: pragma solidity 0.8.16;
OwnableUpgradeable.sol: pragma solidity 0.8.16;
IVRFNFTRandomDrawFactory.sol: pragma solidity 0.8.16;
IOwnableUpgradeable.sol: pragma solidity 0.8.16;
IVRFNFTRandomDraw.sol: pragma solidity 0.8.16;

Recommended Mitigation Steps

Use the same solidity versions
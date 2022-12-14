**src/VRFNFTRandomDrawFactory.sol**
- L4 - IERC721EnumerableUpgradeable is imported but never used, so it should be removed.


**src/VRFNFTRandomDraw.sol**
- L8 - VRFCoordinatorV2 is imported but never used, so it should be removed.

- L22/24/26 - There are several constants: callbackGasLimit, minimumRequestConfirmations and wordsRequested that are written in lower case since they are constants and should be in upper case as in L29/31/33.
They are constant, but they are defined as immutable, I can't find a reason for this to be so.


**src/interfaces/IVRFNFTRandomDraw.sol**
- L20 - The SUPPLY_TOKENS_COUNT_WRONG error is created but never used, so it should be removed.

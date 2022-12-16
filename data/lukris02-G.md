# Gas Optimizations Report for Forgeries contest
## Overview
During the audit, 2 gas issues were found.  

№ | Title | Instance Count
--- | --- | --- 
G-1 | Use unchecked blocks for subtractions where underflow is impossible | 1
G-2 | Elements that are smaller than 32 bytes (256 bits) may increase gas usage | 4 

## Gas Optimizations Findings(2)
### G-1. Use unchecked blocks for subtractions where underflow is impossible
##### Description
In Solidity 0.8+, there’s a default overflow and underflow check on unsigned integers. When an overflow or underflow isn’t possible (after require or if-statement), some gas can be saved by using [unchecked blocks](https://docs.soliditylang.org/en/v0.8.17/control-structures.html#checked-or-unchecked-arithmetic).
##### Instances
- [```_settings.drawingTokenEndId - _settings.drawingTokenStartId < 2```](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L114)

##### Saved
This saves ~35.  
#
### G-2. Elements that are smaller than 32 bytes (256 bits) may increase gas usage
##### Description
According to [docs](https://docs.soliditylang.org/en/v0.8.16/internals/layout_in_storage.html#layout-of-state-variables-in-storage), when using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
##### Instances
- [```uint32 private immutable __version;```](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol#L5) 
- [```uint32 immutable callbackGasLimit = 200_000;```](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L22) 
- [```uint16 immutable minimumRequestConfirmations = 3;```](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L24) 
- [```uint16 immutable wordsRequested = 1;```](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L26) 

##### Recommendation
Consider using a larger size where needed.
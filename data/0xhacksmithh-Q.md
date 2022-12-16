
### [Low-01] Two different solidity version used in this project
```solidity
Following using 0.8.16

File:   src/VRFNFTRandomDrawFactoryProxy.sol
File:   src/VRFNFTRandomDrawFactory.sol
File:   src/VRFNFTRandomDraw.sol 
File:   src/ownable/OwnableUpgradeable.sol

and 3 interface contracts
``` 

```solidity
This contract using ^0.8.0
File:  src/utils/Version.sol
```

Recommended to use a single type, locked and stable version of solidity



### [Low-02] Absence of checking zero address while assigning value to state variables

*Instances (1)*
```solidity
File:   src/VRFNFTRandomDrawFactoryProxy.sol
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactoryProxy.sol#L12
```

### [NC-01] Blank code(code doing nothing) can be removed from contract
*Instances (1)*
```solidity
File:   src/VRFNFTRandomDrawFactory.sol
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L55
```

### [NC-02] State variables should be ```CONSTANT``` rather than ```immutable```
*Instances (6)*
```solidity
File:   src/VRFNFTRandomDraw.sol

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L22
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L24
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L26

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L29
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L31
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L33
```

### [NC-03] State variable(CONSTANT Or Immutable type) should be a constant value instead of a expression 
 *Instances (3)*
```solidity
File:   src/VRFNFTRandomDraw.sol
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L29
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L31
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L33
```

### [NC-04] Instead of large number with zeros, try to use scientific notation that will increase the readability
 *Instances (1)*
```solidity
File:   src/VRFNFTRandomDraw.sol
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L22
```
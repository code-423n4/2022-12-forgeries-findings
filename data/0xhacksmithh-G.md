### [Gas-01] ```public``` Functions those could be ```external```
*Instances (7)*
```solidity
File:   src/ownable/OwnableUpgradeable.sol

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L68
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L73
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L79
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L102
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L114
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L119
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L124
```


### [Gas-02] state variable should be cached rather than re-reading
```
{
        currentChainlinkRequestId = request.currentChainlinkRequestId;  
        hasChosenRandomNumber = request.hasChosenRandomNumber;  
        drawTimelock = request.drawTimelock;  // @audit
    }
```
Here ```request``` should be cached to stack memory rather than re-reading everytime 
*Instances (1)*
```solidity
File:   src/VRFNFTRandomDraw.sol
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L66-L70
```

### [Gas-03] Some state variable could be private instead of public that will help in saving gases
 *Instances (2)*
```solidity
File:   src/VRFNFTRandomDraw.sol
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L40
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L43
```

### [Gas-04] Instead of > try to use <= that will help in saving gas
 *Instances (5)*
```solidity
File:   src/VRFNFTRandomDraw.sol
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L83
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L86
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L90
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L94
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L153
```
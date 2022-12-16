## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Use uint in place of uint32 for single state variable | 3 |
| [GAS-2](#GAS-2) | Use constant instead of immutable for state variables having fixed value | 6 |
| [GAS-3](#GAS-3) | Use assembly to check for address(0) | 3 |
| [GAS-4](#GAS-4) | Using bool for storage incurs overhead | 1 |


[GAS-1] Use uint in place of uint32 for single state variable.
*Instances (3)*:
```solidity
File: src/utils/Version.sol

5:        uint32 private immutable __version;

```
[Link to code](https://github.com/code-423n4/2022-12-forgeries/tree/main/src/utils/Version.sol)
Based on this, return parameter in contractVersion() and parameter in it's constructor can also be changed to uint.

[GAS-2] Use constant instead of immutable for state variables having fixed value
*constants are cheaper than immutable*
*Instances (6)*:
```solidity
File: src/VRFNFTRandomDraw.sol

22:        uint32 immutable callbackGasLimit = 200_000;

24:        uint16 immutable minimumRequestConfirmations = 3;

26:        uint16 immutable wordsRequested = 1;

29:        uint256 immutable HOUR_IN_SECONDS = 60 * 60;

31:        uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);

33:        uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;

```
[Link to code](https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol)

[GAS-3] Use assembly to check for address(0)
*Saves 6 gas per instance*
*Instances (3)*:
```solidity
File: src/VRFNFTRandomDrawFactory.sol

25:        if (_implementation == address(0)) {

```
```solidity
File: src/ownable/OwnableUpgradeable.sol

25:         if (check == address(0)) {

95:         if (_pendingOwner != address(0)) {

```

[GAS-4] Using bool for storage incurs overhead.
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. 
*Instances (1)*:
```solidity
File: src/interfaces/IVRFNFTRandomDraw.sol

65:        bool hasChosenRandomNumber;

```
[Link to code](https://github.com/code-423n4/2022-12-forgeries/tree/main/src/interfaces/IVRFNFTRandomDraw.sol)


### [L01] Missing checks for `address(0x0)` when assigning values to `address` state variables


#### Findings:
```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::51 => coordinator = _coordinator;
2022-12-forgeries/src/VRFNFTRandomDraw.sol::80 => settings = _settings;
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::28 => implementation = _implementation;
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::62 => _owner = _initialOwner;
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::93 => _owner = _newOwner;
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::107 => _pendingOwner = _newOwner;
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::122 => _owner = _pendingOwner;
2022-12-forgeries/src/utils/Version.sol::14 => __version = version;
```



### [L02] `initialize` functions can be front-run

#### Impact
See [this](https://github.com/code-423n4/2021-10-badgerdao-findings/issues/40) finding from a prior badger-dao contest for details
#### Findings:
```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::49 => initializer
2022-12-forgeries/src/VRFNFTRandomDraw.sol::75 => function initialize(address admin, Settings memory _settings)
2022-12-forgeries/src/VRFNFTRandomDraw.sol::77 => initializer
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::24 => constructor(address _implementation) initializer {
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::31 => function initialize(address _initialOwner) initializer external {
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::46 => IVRFNFTRandomDraw(newDrawing).initialize(admin, settings);
2022-12-forgeries/src/interfaces/IVRFNFTRandomDraw.sol::95 => function initialize(address admin, Settings memory _settings) external;
2022-12-forgeries/src/interfaces/IVRFNFTRandomDrawFactory.sol::15 => function initialize(address _initialOwner) external;
```



### [L03] Unspecific Compiler Version Pragma

#### Impact
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.
#### Findings:
```
2022-12-forgeries/src/utils/Version.sol::2 => pragma solidity ^0.8.10;
```



##### `Non-Critical Issues`

### [L01] Adding a return statement when the function defines a named return variable, is redundant


#### Findings:
```
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::50 => return newDrawing;
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::69 => return _owner;
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::74 => return _pendingOwner;
2022-12-forgeries/src/utils/Version.sol::10 => return __version;
```



### [L02] constants should be defined rather than using magic numbers


#### Findings:
```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::29 => uint256 immutable HOUR_IN_SECONDS = 60 * 60;
2022-12-forgeries/src/VRFNFTRandomDraw.sol::31 => uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
2022-12-forgeries/src/VRFNFTRandomDraw.sol::33 => uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
2022-12-forgeries/src/VRFNFTRandomDraw.sol::95 => block.timestamp + (MONTH_IN_SECONDS * 12)
```




### [L03] Event is missing `indexed` fields

#### Impact
Each event should use three indexed fields if there are three or more fields
#### Findings:
```
2022-12-forgeries/src/interfaces/IVRFNFTRandomDrawFactory.sol::11 => event SetupNewDrawing(address user, address drawing);
```


### [L04] Unused file



#### Findings:
```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::1 => // SPDX-License-Identifier: MIT
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::1 => // SPDX-License-Identifier: MIT
2022-12-forgeries/src/VRFNFTRandomDrawFactoryProxy.sol::1 => // SPDX-License-Identifier: MIT
2022-12-forgeries/src/interfaces/IVRFNFTRandomDraw.sol::1 => // SPDX-License-Identifier: MIT
2022-12-forgeries/src/interfaces/IVRFNFTRandomDrawFactory.sol::1 => // SPDX-License-Identifier: MIT
2022-12-forgeries/src/ownable/IOwnableUpgradeable.sol::1 => // SPDX-License-Identifier: MIT
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::1 => // SPDX-License-Identifier: MIT
2022-12-forgeries/src/utils/Version.sol::1 => // SPDX-License-Identifier: MIT
```




### [L05] `public` functions not called by the contract should be declared `external` instead

#### Impact
Contracts are allowed to override their parents’ functions and change the visibility from public to external .
#### Findings:
```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::264 => function hasUserWon(address user) public view returns (bool) {
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::68 => function owner() public view virtual returns (address) {
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::73 => function pendingOwner() public view returns (address) {
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::114 => function resignOwnership() public onlyOwner {
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::119 => function acceptOwnership() public onlyPendingOwner {
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::128 => function cancelOwnershipTransfer() public onlyOwner {
```





### [L06] Interfaces should be moved to separate files

#### Impact
Most of the other interfaces in this project are in their own file in
 the interfaces directory. 
#### Findings:
```
2022-12-forgeries/src/ownable/IOwnableUpgradeable.sol::7 => interface IOwnableUpgradeable {
```








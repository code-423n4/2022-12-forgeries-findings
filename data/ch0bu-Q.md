## 1. Non-library/interface files should use fixed compiler versions, not floating ones

In the contracts, floating pragmas should not be used. Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

```
2	pragma solidity ^0.8.10;
```
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol


## 2. Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions

See the following link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

- https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps

```
12	abstract contract OwnableUpgradeable is IOwnableUpgradeable, Initializable {
```
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol


## 3. Add constructor initializers

As per OpenZeppelin’s (OZ) [recommendation](https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/6), “The guidelines are now to make it impossible for anyone to run initialize on an implementation contract, by adding an empty constructor with the initializer modifier. So the implementation contract gets initialized automatically upon deployment.”

```
95	function initialize(address admin, Settings memory _settings) external;
```
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol
```
15	function initialize(address _initialOwner) external;
```
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDrawFactory.sol
```
75       function initialize(address admin, Settings memory _settings)
```
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol
```
31       function initialize(address _initialOwner) initializer external {
```
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol


## 4. State variables can be set as `constant` instead of `immutable`
Difference between two is that `constant` variables can never be changed after compilation, while `immutable` variables can be set within the constructor. 

Variables in this contract are not set inside a constructor and will never have to be changed, thus can be set as `constant`s.

```
22	uint32 immutable callbackGasLimit = 200_000;
24	uint16 immutable minimumRequestConfirmations = 3;
26	uint16 immutable wordsRequested = 1;
29	uint256 immutable HOUR_IN_SECONDS = 60 * 60;
31	uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
33	uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol


## 5. State variable visibility is not set

It is best practice to set visibility for state variables.

```
22	uint32 immutable callbackGasLimit = 200_000;
24	uint16 immutable minimumRequestConfirmations = 3;
26	uint16 immutable wordsRequested = 1;
29	uint256 immutable HOUR_IN_SECONDS = 60 * 60;
31	uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
33	uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
37	VRFCoordinatorV2Interface immutable coordinator;
```
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol

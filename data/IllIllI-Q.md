

## Summary

### Low Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | Use `Ownable2Step` rather than `Ownable` | 2 | 

Total: 2 instances over 1 issues


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions | 2 | 
| [N&#x2011;02] | `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings | 1 | 
| [N&#x2011;03] | `constant`s should be defined rather than using magic numbers | 1 | 
| [N&#x2011;04] | Numeric values having to do with time should use time units for readability | 4 | 
| [N&#x2011;05] | Lines are too long | 1 | 
| [N&#x2011;06] | Non-library/interface files should use fixed compiler versions, not floating ones | 1 | 
| [N&#x2011;07] | NatSpec is incomplete | 4 | 
| [N&#x2011;08] | Contracts should have full test coverage | 1 | 
| [N&#x2011;09] | Large or complicated code bases should implement fuzzing tests | 1 | 

Total: 16 instances over 9 issues





## Low Risk Issues

### [L&#x2011;01]  Use `Ownable2Step` rather than `Ownable`
[`Ownable2Step`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/3d7a93876a2e5e1d7fe29b5a0e96e222afdc4cfa/contracts/access/Ownable2Step.sol#L31-L56) and [`Ownable2StepUpgradeable`](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/25aabd286e002a1526c345c8db259d57bdf0ad28/contracts/access/Ownable2StepUpgradeable.sol#L47-L63) prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own.

*There are 2 instances of this issue:*

```solidity
File: src/VRFNFTRandomDrawFactory.sol

14    contract VRFNFTRandomDrawFactory is
15        IVRFNFTRandomDrawFactory,
16        OwnableUpgradeable,
17        UUPSUpgradeable,
18:       Version(1)

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L14-L18

```solidity
File: src/VRFNFTRandomDraw.sol

15    contract VRFNFTRandomDraw is
16        IVRFNFTRandomDraw,
17        VRFConsumerBaseV2,
18        OwnableUpgradeable,
19:       Version(1)

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L15-L19

## Non-critical Issues

### [N&#x2011;01]  Upgradeable contract is missing a `__gap[50]` storage variable to allow for new storage variables in later versions
See [this](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps) link for a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

*There are 2 instances of this issue:*

```solidity
File: src/VRFNFTRandomDrawFactory.sol

14    contract VRFNFTRandomDrawFactory is
15        IVRFNFTRandomDrawFactory,
16        OwnableUpgradeable,
17        UUPSUpgradeable,
18:       Version(1)

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L14-L18

```solidity
File: src/VRFNFTRandomDraw.sol

15    contract VRFNFTRandomDraw is
16        IVRFNFTRandomDraw,
17        VRFConsumerBaseV2,
18        OwnableUpgradeable,
19:       Version(1)

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L15-L19

### [N&#x2011;02]  `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings

*There is 1 instance of this issue:*

```solidity
File: src/VRFNFTRandomDrawFactory.sol

55:       function _authorizeUpgrade(address newImplementation)

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L55

### [N&#x2011;03]  `constant`s should be defined rather than using magic numbers
Even [assembly](https://github.com/code-423n4/2022-05-opensea-seaport/blob/9d7ce4d08bf3c3010304a0476a785c70c0e90ae7/contracts/lib/TokenTransferrer.sol#L35-L39) can benefit from using readable constants instead of hex/numeric literals

*There is 1 instance of this issue:*

```solidity
File: src/VRFNFTRandomDraw.sol

/// @audit 12
95:               block.timestamp + (MONTH_IN_SECONDS * 12)

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L95

### [N&#x2011;04]  Numeric values having to do with time should use time units for readability
There are [units](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#time-units) for seconds, minutes, hours, days, and weeks, and since they're defined, they should be used

*There are 4 instances of this issue:*

```solidity
File: src/VRFNFTRandomDraw.sol

/// @audit 60
/// @audit 60
29:       uint256 immutable HOUR_IN_SECONDS = 60 * 60;

/// @audit 3600
31:       uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);

/// @audit 3600
33:       uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L29

### [N&#x2011;05]  Lines are too long
Usually lines in source code are limited to [80](https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width) characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over [164](https://github.com/aizatto/character-length) characters, the lines below should be split when they reach that length

*There is 1 instance of this issue:*

```solidity
File: src/interfaces/IVRFNFTRandomDraw.sol

64:           /// @notice has chosen a random number (in case random number = 0(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0))

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L64

### [N&#x2011;06]  Non-library/interface files should use fixed compiler versions, not floating ones

*There is 1 instance of this issue:*

```solidity
File: src/utils/Version.sol

2:    pragma solidity ^0.8.10;

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/utils/Version.sol#L2

### [N&#x2011;07]  NatSpec is incomplete

*There are 4 instances of this issue:*

```solidity
File: src/interfaces/IVRFNFTRandomDrawFactory.sol

/// @audit Missing: '@return'
20        /// @notice Function to make a new drawing
21        /// @param settings settings for the new drawing
22        function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)
23            external
24:           returns (address);

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDrawFactory.sol#L20-L24

```solidity
File: src/interfaces/IVRFNFTRandomDraw.sol

/// @audit Missing: '@return'
106       /// @notice Function to determine if the user has won in the current drawing
107       /// @param user address for the user to check if they have won in the current drawing
108:      function hasUserWon(address user) external view returns (bool);

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L106-L108

```solidity
File: src/VRFNFTRandomDrawFactory.sol

/// @audit Missing: '@return'
36        /// @notice Function to make a new drawing
37        /// @param settings settings for the new drawing
38        function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)
39            external
40:           returns (address)

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L36-L40

```solidity
File: src/VRFNFTRandomDraw.sol

/// @audit Missing: '@return'
262       /// @notice Function to determine if the user has won in the current drawing
263       /// @param user address for the user to check if they have won in the current drawing
264:      function hasUserWon(address user) public view returns (bool) {

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L262-L264

### [N&#x2011;08]  Contracts should have full test coverage
While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

*There is 1 instance of this issue:*

```solidity
File: src/interfaces/IVRFNFTRandomDrawFactory.sol


```

### [N&#x2011;09]  Large or complicated code bases should implement fuzzing tests
Large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts, should implement [fuzzing tests](https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05). Fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and fuzzers, with properly and extensively-written invariants, can close this testing gap significantly.

*There is 1 instance of this issue:*

```solidity
File: src/interfaces/IVRFNFTRandomDrawFactory.sol


```


___

## Excluded findings
These findings are excluded from awards calculations because there are publicly-available automated tools that find them. The valid ones appear here for completeness

## Summary

### Low Risk Issues
| |Issue|Instances|
|-|:-|:-:|
| [L&#x2011;01] | Missing checks for `address(0x0)` when assigning values to `address` state variables | 3 | 

Total: 3 instances over 1 issues


### Non-critical Issues
| |Issue|Instances|
|-|:-|:-:|
| [N&#x2011;01] | `public` functions not called by the contract should be declared `external` instead | 1 | 
| [N&#x2011;02] | Event is missing `indexed` fields | 4 | 

Total: 5 instances over 2 issues





## Low Risk Issues

### [L&#x2011;01]  Missing checks for `address(0x0)` when assigning values to `address` state variables

*There are 3 instances of this issue:*

```solidity
File: src/ownable/OwnableUpgradeable.sol

/// @audit (valid but excluded finding)
62:           _owner = _initialOwner;

/// @audit (valid but excluded finding)
93:           _owner = _newOwner;

/// @audit (valid but excluded finding)
107:          _pendingOwner = _newOwner;

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/ownable/OwnableUpgradeable.sol#L62

## Non-critical Issues

### [N&#x2011;01]  `public` functions not called by the contract should be declared `external` instead
Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents' functions and change the visibility from `external` to `public`.

*There is 1 instance of this issue:*

```solidity
File: src/VRFNFTRandomDraw.sol

/// @audit (valid but excluded finding)
75        function initialize(address admin, Settings memory _settings)
76            public
77:           initializer

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L75-L77

### [N&#x2011;02]  Event is missing `indexed` fields
Index event fields make the field more quickly accessible [to off-chain tools](https://ethereum.stackexchange.com/questions/40396/can-somebody-please-explain-the-concept-of-event-indexing) that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each `event` should use three `indexed` fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*There are 4 instances of this issue:*

```solidity
File: src/interfaces/IVRFNFTRandomDrawFactory.sol

/// @audit (valid but excluded finding)
11:       event SetupNewDrawing(address user, address drawing);

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDrawFactory.sol#L11

```solidity
File: src/interfaces/IVRFNFTRandomDraw.sol

/// @audit (valid but excluded finding)
43:       event InitializedDraw(address indexed sender, Settings settings);

/// @audit (valid but excluded finding)
45:       event SetupDraw(address indexed sender, Settings settings);

/// @audit (valid but excluded finding)
49:       event DiceRollComplete(address indexed sender, CurrentRequest request);

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L43


# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1      | Mistake when calculating `MONTH_IN_SECONDS` value will allow bigger max values for `_settings.drawBufferTime` and `_settings.recoverTimelock` | Low | 1 |
| 2      | Front-runable `initialize` function | Low | 1 |
| 3      | Immutable state variables lack zero address checks | Low | 1 |
| 4      | Event should be emitted in the `redraw()` function | NC | 1 |
| 5      | Use `constant` instead of `immutable` for variables that are not set in `constructor` | NC | 6 |
| 6      | Use solidity time-based units | NC | 3 |


## Findings

### 1- Mistake when calculating `MONTH_IN_SECONDS` value will allow bigger max values for `_settings.drawBufferTime` and `_settings.recoverTimelock` :

When calculating the value of `MONTH_IN_SECONDS` which is supposed to represent amount of seconds in a month, the formula used contain an error so instead of getting the equivalent of 1 month in seconds the code will calculate the equivalent of 7 months in seconds.

This will allow a bigger maximum value for both `_settings.drawBufferTime` and `_settings.recoverTimelock` variables which can possibly impact the protocol working in the futur.

#### Risk : Low

#### Proof of Concept

The issue occurs here :

File: src/VRFNFTRandomDraw.sol [Line 32-33](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L32-L33)

```
// @dev about 30 days in a month
uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```

The above formula calculate the equivalent of 7 months in seconds as `1 month == 3600 * 24 * 30`

And the `MONTH_IN_SECONDS` is used later in the `initialize` function function to check the max values of the variables `_settings.drawBufferTime` and `_settings.recoverTimelock` :

File: src/VRFNFTRandomDraw.sol [Line 86-88](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L86-L88)

```
if (_settings.drawBufferTime > MONTH_IN_SECONDS) {
    revert REDRAW_TIMELOCK_NEEDS_TO_BE_LESS_THAN_A_MONTH();
}
```

File: src/VRFNFTRandomDraw.sol [Line 93-98](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L93-L98)

```
if (
    _settings.recoverTimelock >
    block.timestamp + (MONTH_IN_SECONDS * 12)
) {
    revert RECOVER_TIMELOCK_NEEDS_TO_BE_LESS_THAN_A_YEAR();
}
```

This will eventuly allow bigger maximum values for both variabales which doesn't have big impact on the current contract logic (can only cause longer wait for redraw or admin NFT reclaim) but it can cause problems if this contract is used in interaction with part of the protocol.

#### Mitigation

The solve this issue correct the formula of `MONTH_IN_SECONDS` as follow :

```
// @dev about 30 days in a month
uint256 immutable MONTH_IN_SECONDS = 3600 * 24 * 30;
```

### 2- Front-runable `initialize` function :

The `initialize` function is used in `VRFNFTRandomDrawFactory` contract to initialize the contract owner but there is the issue that any attacker can initialize the contract before the legitimate deployer and even if the developers when deploying call immediately the `initialize` function, malicious agents can trace the protocol deployment transactions and insert their own transaction between them and by doing so they front run the developers call and gain the ownership of the contract.

The impact of this issue is : 

* In the best case developers notice the problem and have to redeploy the contract and thus costing more gas.

* In the worst case the protocol continue to work with the wrong owner which could lead to various problems for users.

#### Risk : Low 

#### Proof of Concept

Instances include:

File: src/VRFNFTRandomDrawFactory.sol [Line 31-34](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L31-L34)

```
function initialize(address _initialOwner) initializer external {
    __Ownable_init(_initialOwner);
    emit SetupFactory();
}
```

#### Mitigation

It's recommended to use the constructor to initialize non-proxied contracts. But in this case for initializing proxy (upgradable) contracts deploy contracts using a factory contract that immediately calls initialize after deployment or make sure to call it immediately after deployment and verify that the transaction succeeded.


### 3- Immutable state variables lack zero address checks  :

Constructors should check the values written in an immutable state variables(address) is not the address(0)

#### Risk : Low

#### Proof of Concept
Instances include:

File: src/VRFNFTRandomDraw.sol [Line 51](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L51)
```
coordinator = _coordinator;
```

#### Mitigation
Add non-zero address checks in the constructors for the instances aforementioned.


### 4- Event should be emitted in the `redraw()` function :

The `redraw()` function is used to reset the giveaway process in case the current winner doesn't reclaim it's NFT, thus the fonction should emit an event so that Dapps and users can know that a new request has been made to choose a new winner. 

#### Risk : NON CRITICAL

#### Proof of Concept
Instances include:

File: src/VRFNFTRandomDraw.sol

[function redraw() external onlyOwner returns (uint256)](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L203-L225)

#### Mitigation
Emit an event in the `redraw()` function.

### 5- Use `constant` instead of `immutable` for variables that are not set in `constructor` :

In solidity best practices `constant` keyword is used for variables which don't change in value after contract compilation and the `immutable` keyword is used for variables that are set only in the constructor, and as all the `immutable` variables used in VRFNFTRandomDraw are not set in constructor they should be marked `constant`. 

#### Risk : NON CRITICAL

#### Proof of Concept
Instances include:

File: src/VRFNFTRandomDraw.sol [Line 21-33](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L21-L33)

```
/// @notice Our callback is just setting a few variables, 200k should be more than enough gas.
uint32 immutable callbackGasLimit = 200_000;
/// @notice Chainlink request confirmations, left at the default
uint16 immutable minimumRequestConfirmations = 3;
/// @notice Number of words requested in a drawing
uint16 immutable wordsRequested = 1;

/// @dev 60 seconds in a min, 60 mins in an hour
uint256 immutable HOUR_IN_SECONDS = 60 * 60;
/// @dev 24 hours in a day 7 days in a week
uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
// @dev about 30 days in a month
uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```

#### Mitigation

Replace `immutable` by `constant` in the aforementioned variables.

### 6- Use solidity time-based units :

Solidity contains time-based units (seconds, minutes, hours, days and weeks) which should be used when declaring time based variables/constants instead of using literal numbers, this is done for better readability and to avoid mistakes.

#### Risk : NON CRITICAL

#### Proof of Concept
Instances include:

File: src/VRFNFTRandomDraw.sol [Line 28-33](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L28-L33)

```
/// @dev 60 seconds in a min, 60 mins in an hour
uint256 immutable HOUR_IN_SECONDS = 60 * 60;
/// @dev 24 hours in a day 7 days in a week
uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
// @dev about 30 days in a month
uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```

#### Mitigation

Use time units when declaring time-based variables.
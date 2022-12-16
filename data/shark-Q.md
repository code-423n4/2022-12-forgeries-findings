## 1. Unused imports

Contracts that are imported and not used anywhere in the code are most likely an accident due to incomplete refactoring. Such imports can lead to confusion among readers.

Here is an example of this issue:

[File: VRFNFTRandomDraw.sol#L8](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L8)

```solidity
import {VRFCoordinatorV2, VRFCoordinatorV2Interface} from "@chainlink/contracts/src/v0.8/VRFCoordinatorV2.sol";
```

In the example above, `VRFCoordinatorV2` is never used in the contract. As such it can be removed from the import like so:

```solidity
import {VRFCoordinatorV2Interface} from "@chainlink/contracts/src/v0.8/VRFCoordinatorV2.sol";
```

## 2. Empty functions

Empty functions can reduce readability because readers need to guess whether it’s intentional or not. So writing a clear comment for empty functions is a good practice. Additionally, an empty function should also revert on call/emit an event.

- File: `VRFNFTRandomDrawFactory.sol` [Line 55-59](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L55-L59)

## 3. Typos

File: `OwnableUpgradeable.sol` [Line 113](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L113)

```
    /// @audit "callably" -> "callable"
    /// @dev only callably by the owner, dangerous call.
```

File: `IVRFNFTRandomDraw.sol` [Line 46](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L46)

```
    /// @audit "aftr" -> "after"
    /// @notice When the owner reclaims nft aftr recovery time delay
```

## 4. Empty Event

The event below has no parameters in it to emit anything. Consider either removing this event or refactoring it with relevant info.

File: `IVRFNFTRandomDrawFactory.sol` [Line 18](https://github.com/code-423n4/2022-12-forgeries/blob/82a5fc7540249325ae48498720d40d943ee17e27/src/interfaces/IVRFNFTRandomDrawFactory.sol#L18)

```solidity
    event SetupFactory();
```

## 5. Limit line length

To comply with the [Solidity Style Guide](https://docs.soliditylang.org/en/develop/style-guide.html#maximum-line-length), lines should be limited to **120** characters max.

For example, the instance listed below is way over the limit of 120 characters:

File: `IVRFNFTRandomDraw.sol` [Line 64](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L64)

## 6. Unspecific Pragma Version

Locking the pragma helps ensure that contracts don't get deployed with unintended versions, for example, the latest compiler version which could have higher risks of undiscovered bugs.

For example, The pragma version in [`Version.sol`](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol#L2) is unspecific (`pragma solidity ^0.8.10;`).

## 7. No storage gap for upgradeable contracts

For upgradeable contracts, there should be a storage gap at the end of the contract. Without a storage gap, Storage clashes could occur while using inheritance.

For example:

File: [`VRFNFTRandomDrawFactory.sol`](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol)

Consider adding the variable `__gap` at the end of `VRFNFTRandomDrawFactory.sol`:

```
uint256[50] private __gap;
```

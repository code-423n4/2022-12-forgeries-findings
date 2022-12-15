- [Low](#low)
    - [**1. The owner can remove the prize**](#1-the-owner-can-remove-the-prize)
    - [**2. Lack of whitelist**](#2-lack-of-whitelist)
    - [**3. Timestamp dependence**](#3-timestamp-dependence)
    - [**4. Contracts without GAP can lead a upgrade disaster**](#4-contracts-without-gap-can-lead-a-upgrade-disaster)
    - [**5. Incorrect logic according to error messages**](#5-incorrect-logic-according-to-error-messages)
- [Non critical](#non-critical)
    - [**6. Use abstract for base contracts**](#6-use-abstract-for-base-contracts)
    - [**7. Typo error**](#7-typo-error)
    - [**8. Outdated compiler**](#8-outdated-compiler)
    - [**9. Improve NatSpec**](#9-improve-natspec)
    - [**10. Use solidity literals instead of math**](#10-use-solidity-literals-instead-of-math)

# Low

## **1. The owner can remove the prize**

The design of the contract allows it to be raffled again, or even for the admin to claim it after a time that is configurable, and it can be brief, it does not make sense that the admin is not forced to send the token to the winner, in the case of that the winner does not make said call, since he may be indisposed, and not for that reason lose his draw. Also, since the admin is going to make the gas payment for the transfer, it doesn't make sense for the admin to receive it and not the winner.

**Affected source code:**

- [VRFNFTRandomDraw.sol:277](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L277)
- [VRFNFTRandomDraw.sol:306](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L306)

## **2. Lack of whitelist**

There is no token whitelist, so anyone can create an NFT token that they later destroy, or change ownership at the will of the contract owner. So it can be used to force people to buy a certain token (`settings.drawingToken`) and receive nothing in return.

## **3. Timestamp dependence**

There are three main considerations when using a timestamp to execute a critical function in a contract, especially when actions involve fund transfer.

- Timestamp manipulation
  - Be aware that the timestamp of the block can be manipulated by a miner
- The 15-second Rule
  - The [Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) (Ethereum’s reference specification) does not specify a constraint on how much blocks can drift in time, but [it does specify](https://ethereum.stackexchange.com/a/5926/46821) that each timestamp should be bigger than the timestamp of its parent. Popular Ethereum protocol implementations Geth and Parity both reject blocks with timestamp more than 15 seconds in future. Therefore, a good rule of thumb in evaluating timestamp usage is: if the scale of your time-dependent event can vary by 15 seconds and maintain integrity, it is safe to use a `block.timestamp`.
- Avoid using `block.number` as a timestamp
  - It is possible to estimate a time delta using the `block.number` property and [average block time](https://etherscan.io/chart/blocktime), however this is not future proof as block times may change (such as [fork reorganisations](https://blog.ethereum.org/2015/08/08/chain-reorganisation-depth-expectations) and the [difficulty bomb](https://github.com/ethereum/EIPs/issues/649)). In a sale spanning days, the 15-second rule allows one to achieve a more reliable estimate of time.
  See [SWC-116](https://swcregistry.io/docs/SWC-116)

**Reference:**

- https://consensys.net/blog/developers/solidity-best-practices-for-smart-contract-security/

**Affected source code:**

- [VRFNFTRandomDraw.sol:306](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L306)
- [VRFNFTRandomDraw.sol:204](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L204)

## **4. Contracts without GAP can lead a upgrade disaster**

Some contracts do not implement good upgradeable logic.
Upgrading a contract will need a `__gap` storage in order to avoid storage collisions.

Proxied contracts MUST include an empty reserved space in storage, usually named `__gap`, in all the inherited contracts.

For example, the contract `Version` or `VRFConsumerBaseV2` are inherited by upgradeable contracts, but these contracts don't have a `__gap` storage, so if a new variable is created it could lead to storage collisions that will produce a contract disaster, including loss of funds.

**Reference:**
- https://docs.openzeppelin.com/contracts/3.x/upgradeable#storage_gaps
- https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/2cb8996b777060e658e2b8c9b1630313aedb04c0/contracts/token/ERC20/ERC20Upgradeable.sol#L394

**Affected source code:**

- [Version.sol:4](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/utils/Version.sol#L4)
- [VRFNFTRandomDrawFactory.sol:14](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L14)
- [VRFNFTRandomDraw.sol:15](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L15)
- [VRFNFTRandomDraw.sol:17](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L17)

**Recommended Mitigation Steps:**

- Add `uint256[50] private __gap;` to all upgradable contracts.

## **5. Incorrect logic according to error messages**

The error messages indicate that it has to be greater than, but in the case of equals, it would not fail. It is recommended to correct the error message, or the logic.

```diff
        // Check values in memory:
-       if (_settings.drawBufferTime < HOUR_IN_SECONDS) {
+       if (_settings.drawBufferTime <= HOUR_IN_SECONDS) {
            revert REDRAW_TIMELOCK_NEEDS_TO_BE_MORE_THAN_AN_HOUR();
        }
-       if (_settings.drawBufferTime > MONTH_IN_SECONDS) {
+       if (_settings.drawBufferTime >= MONTH_IN_SECONDS) {
            revert REDRAW_TIMELOCK_NEEDS_TO_BE_LESS_THAN_A_MONTH();
        }
```

**Affected source code:**

- [VRFNFTRandomDraw.sol:82-88](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L82-L88)

---

# Non critical

## **6. Use `abstract` for base contracts**

`abstract` contracts are contracts that have at least one function without its implementation. **An instance of an abstract cannot be created.**

**Reference:**

- https://docs.soliditylang.org/en/v0.6.2/contracts.html#abstract-contracts

**Affected source code:**

- [Version.sol:4](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/utils/Version.sol#L4)

## **7. Typo error**

There is a typo error in the following line:

> // Transfer token to the winter.

It must be `winner`.

**Affected source code:**

- [VRFNFTRandomDraw.sol:294](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L294)

## **8. Outdated compiler**

The pragma version used is:

```
pragma solidity 0.8.16;
```

The minimum required version must be [0.8.17](https://github.com/ethereum/solidity/releases/tag/v0.8.17); otherwise, contracts will be affected by the following **important bug fixes**:

[0.8.17](https://blog.soliditylang.org/2022/09/08/solidity-0.8.17-release-announcement/)

- Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.

Apart from these, there are several minor bug fixes and improvements.

## **9. Improve NatSpec**

Solidity contracts can use a special form of comments to provide rich documentation for functions, return variables and more. This special form is named the Ethereum Natural Language Specification Format (NatSpec).

It was detected that some methods doesn't added the `return` documentation, like the following ones:

- `makeNewDraw` in [VRFNFTRandomDrawFactory.sol:38](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L38) and [IVRFNFTRandomDrawFactory.sol:22](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDrawFactory.sol#L22):

```diff
    /// @notice Function to make a new drawing
    /// @param settings settings for the new drawing
+   /// @return new draw address
    function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)
        external
        returns (address)
    {
```

- `hasUserWon` in [IVRFNFTRandomDraw.sol:108](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L108) and [VRFNFTRandomDraw.sol:264](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L264):

```diff
    /// @notice Function to determine if the user has won in the current drawing
    /// @param user address for the user to check if they have won in the current drawing
+   /// @return tue if the user won
    function hasUserWon(address user) external view returns (bool);
```

**Reference:**

- https://docs.soliditylang.org/en/develop/natspec-format.html

## **10. Use solidity literals instead of math**

Suffixes like `seconds`, `minutes`, `hours`, `days` and `weeks` after literal numbers can be used to specify units of time where seconds are the base unit and units are considered naively in the following way:

- 1 == 1 seconds
- 1 minutes == 60 seconds
- 1 hours == 60 minutes
- 1 days == 24 hours
- 1 weeks == 7 days
- MONTH_IN_SECONDS * 12 == 365 days

**Reference:**

- https://docs.soliditylang.org/en/v0.8.14/units-and-global-variables.html#time-units

```diff
    /// @dev 60 seconds in a min, 60 mins in an hour
-   uint256 immutable HOUR_IN_SECONDS = 60 * 60;
+   uint256 immutable HOUR_IN_SECONDS = 1 hours;
    /// @dev 24 hours in a day 7 days in a week
-   uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
+   uint256 immutable WEEK_IN_SECONDS = 1 weeks;
    // @dev about 30 days in a month
-   uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
+   uint256 immutable MONTH_IN_SECONDS = 30 days;
```

**Affected source code:**

- [VRFNFTRandomDraw.sol:28-33](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L28-L33)

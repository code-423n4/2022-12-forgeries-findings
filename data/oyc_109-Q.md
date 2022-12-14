
## [L-01] Unspecific Compiler Version Pragma

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

```
2022-12-forgeries/src/utils/Version.sol::2 => pragma solidity ^0.8.10;
```

## [L-02] Use of Block.timestamp

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.

```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::57 => /// @return drawTimelock block.timestamp when a redraw can be issued
2022-12-forgeries/src/VRFNFTRandomDraw.sol::90 => if (_settings.recoverTimelock < block.timestamp + WEEK_IN_SECONDS) {
2022-12-forgeries/src/VRFNFTRandomDraw.sol::95 => block.timestamp + (MONTH_IN_SECONDS * 12)
2022-12-forgeries/src/VRFNFTRandomDraw.sol::153 => request.drawTimelock > block.timestamp
2022-12-forgeries/src/VRFNFTRandomDraw.sol::159 => request.drawTimelock = block.timestamp + settings.drawBufferTime;
2022-12-forgeries/src/VRFNFTRandomDraw.sol::204 => if (request.drawTimelock >= block.timestamp) {
2022-12-forgeries/src/VRFNFTRandomDraw.sol::306 => if (settings.recoverTimelock > block.timestamp) {
```

## [L-03] Unsafe use of transfer()/transferFrom() with IERC20

Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)'s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert. Use OpenZeppelin’s SafeERC20's safeTransfer()/safeTransferFrom() instead

```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::187 => IERC721EnumerableUpgradeable(settings.token).transferFrom(
2022-12-forgeries/src/VRFNFTRandomDraw.sol::295 => IERC721EnumerableUpgradeable(settings.token).transferFrom(
2022-12-forgeries/src/VRFNFTRandomDraw.sol::315 => IERC721EnumerableUpgradeable(settings.token).transferFrom(
```

## [L-04] Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions

__gap is empty reserved space in storage that is recommended to be put in place in upgradeable contracts. It allows new state variables to be added in the future without compromising the storage compatibility with existing deployments

```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::5 => import {IERC721EnumerableUpgradeable} from "@openzeppelin/contracts-upgradeable/token/ERC721/extensions/IERC721EnumerableUpgradeable.sol";
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::4 => import {IERC721EnumerableUpgradeable} from "@openzeppelin/contracts-upgradeable/token/ERC721/extensions/IERC721EnumerableUpgradeable.sol";
```

## [N-01] Use a more recent version of solidity

Use a solidity version of at least 0.8.4 to get bytes.concat() instead of abi.encodePacked(<bytes>,<bytes>)
Use a solidity version of at least 0.8.12 to get string.concat() instead of abi.encodePacked(<str>,<str>)
Use a solidity version of at least 0.8.13 to get the ability to use using for with a list of free functions

```
2022-12-forgeries/src/utils/Version.sol::2 => pragma solidity ^0.8.10;
```


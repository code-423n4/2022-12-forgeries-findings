# c4udit Report

## Files analyzed
- examples\VRFNFTRandomDraw.sol
- examples\VRFNFTRandomDrawFactory.sol
- examples\VRFNFTRandomDrawFactoryProxy.sol
- examples\interfaces\IVRFNFTRandomDraw.sol
- examples\interfaces\IVRFNFTRandomDrawFactory.sol
- examples\ownable\IOwnableUpgradeable.sol
- examples\ownable\OwnableUpgradeable.sol
- examples\utils\Version.sol

## Issues found

### Cache Array Length Outside of Loop

#### Impact
Issue Information: [G002](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g002---cache-array-length-outside-of-loop)

#### Findings:
```
examples\VRFNFTRandomDraw.sol::101 => if (_settings.token.code.length == 0) {
examples\VRFNFTRandomDraw.sol::106 => if (_settings.drawingToken.code.length == 0) {
examples\VRFNFTRandomDraw.sol::241 => if (_randomWords.length != wordsRequested) {
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Long Revert Strings

#### Impact
Issue Information: [G007](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g007---long-revert-strings)

#### Findings:
```
examples\VRFNFTRandomDraw.sol::5 => import {IERC721EnumerableUpgradeable} from "@openzeppelin/contracts-upgradeable/token/ERC721/extensions/IERC721EnumerableUpgradeable.sol";
examples\VRFNFTRandomDraw.sol::7 => import {VRFConsumerBaseV2} from "@chainlink/contracts/src/v0.8/VRFConsumerBaseV2.sol";
examples\VRFNFTRandomDraw.sol::8 => import {VRFCoordinatorV2, VRFCoordinatorV2Interface} from "@chainlink/contracts/src/v0.8/VRFCoordinatorV2.sol";
examples\VRFNFTRandomDraw.sol::10 => import {IVRFNFTRandomDraw} from "./interfaces/IVRFNFTRandomDraw.sol";
examples\VRFNFTRandomDrawFactory.sol::4 => import {IERC721EnumerableUpgradeable} from "@openzeppelin/contracts-upgradeable/token/ERC721/extensions/IERC721EnumerableUpgradeable.sol";
examples\VRFNFTRandomDrawFactory.sol::5 => import {ClonesUpgradeable} from "@openzeppelin/contracts-upgradeable/proxy/ClonesUpgradeable.sol";
examples\VRFNFTRandomDrawFactory.sol::6 => import {UUPSUpgradeable} from "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
examples\VRFNFTRandomDrawFactory.sol::8 => import {IVRFNFTRandomDrawFactory} from "./interfaces/IVRFNFTRandomDrawFactory.sol";
examples\VRFNFTRandomDrawFactory.sol::9 => import {IVRFNFTRandomDraw} from "./interfaces/IVRFNFTRandomDraw.sol";
examples\VRFNFTRandomDrawFactoryProxy.sol::4 => import {ERC1967Proxy} from "@openzeppelin/contracts/proxy/ERC1967/ERC1967Proxy.sol";
examples\VRFNFTRandomDrawFactoryProxy.sol::5 => import {IVRFNFTRandomDrawFactory} from "./interfaces/IVRFNFTRandomDrawFactory.sol";
examples\ownable\OwnableUpgradeable.sol::5 => import {Initializable} from "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use Shift Right/Left instead of Division/Multiplication if possible

#### Impact
Issue Information: [G008](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md/#g008---use-shift-rightleft-instead-of-divisionmultiplication-if-possible)

#### Findings:
```
examples\VRFNFTRandomDraw.sol::31 => uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
examples\VRFNFTRandomDraw.sol::33 => uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)
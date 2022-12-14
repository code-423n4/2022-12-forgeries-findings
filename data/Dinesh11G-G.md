### 1st Bug
Cache Array Length Outside of Loop

#### Impact
Issue Information: 
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

Example
🤦 Bad:

for (uint256 i = 0; i < array.length; i++) {
    // invariant: array's length is not changed
}
🚀 Good:

uint256 len = array.length
for (uint256 i = 0; i < len; i++) {
    // invariant: array's length is not changed
}

#### Findings:
```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::101 => if (_settings.token.code.length == 0) {
2022-12-forgeries/src/VRFNFTRandomDraw.sol::106 => if (_settings.drawingToken.code.length == 0) {
2022-12-forgeries/src/VRFNFTRandomDraw.sol::241 => if (_randomWords.length != wordsRequested) {
```
#### Tools used
Manual code review

### 2nd Bug
Long Revert Strings

#### Impact
Issue Information: 
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.

Example
🤦 Bad:

require(condition, "UniswapV3: The reentrancy guard. A transaction cannot re-enter the pool mid-swap");
🚀 Good (with shorter string):

// TODO: Provide link to a reference of error codes
require(condition, "LOK");
🚀 Good (with custom errors):

/// @notice A transaction cannot re-enter the pool mid-swap.
error NoReentrancy();

// ...

if (!condition) {
    revert NoReentrancy();
}

#### Findings:
```
2022-12-forgeries/script/SetupContractsScript.s.sol::8 => import {VRFNFTRandomDrawFactory} from "../src/VRFNFTRandomDrawFactory.sol";
2022-12-forgeries/script/SetupContractsScript.s.sol::9 => import {VRFNFTRandomDrawFactoryProxy} from "../src/VRFNFTRandomDrawFactoryProxy.sol";
2022-12-forgeries/script/SetupContractsScript.s.sol::10 => import {VRFCoordinatorV2Interface} from "@chainlink/contracts/src/v0.8/VRFCoordinatorV2.sol";
2022-12-forgeries/src/VRFNFTRandomDraw.sol::5 => import {IERC721EnumerableUpgradeable} from "@openzeppelin/contracts-upgradeable/token/ERC721/extensions/IERC721EnumerableUpgradeable.sol";
2022-12-forgeries/src/VRFNFTRandomDraw.sol::7 => import {VRFConsumerBaseV2} from "@chainlink/contracts/src/v0.8/VRFConsumerBaseV2.sol";
2022-12-forgeries/src/VRFNFTRandomDraw.sol::8 => import {VRFCoordinatorV2, VRFCoordinatorV2Interface} from "@chainlink/contracts/src/v0.8/VRFCoordinatorV2.sol";
2022-12-forgeries/src/VRFNFTRandomDraw.sol::10 => import {IVRFNFTRandomDraw} from "./interfaces/IVRFNFTRandomDraw.sol";
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::4 => import {IERC721EnumerableUpgradeable} from "@openzeppelin/contracts-upgradeable/token/ERC721/extensions/IERC721EnumerableUpgradeable.sol";
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::5 => import {ClonesUpgradeable} from "@openzeppelin/contracts-upgradeable/proxy/ClonesUpgradeable.sol";
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::6 => import {UUPSUpgradeable} from "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::8 => import {IVRFNFTRandomDrawFactory} from "./interfaces/IVRFNFTRandomDrawFactory.sol";
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::9 => import {IVRFNFTRandomDraw} from "./interfaces/IVRFNFTRandomDraw.sol";
2022-12-forgeries/src/VRFNFTRandomDrawFactoryProxy.sol::4 => import {ERC1967Proxy} from "@openzeppelin/contracts/proxy/ERC1967/ERC1967Proxy.sol";
2022-12-forgeries/src/VRFNFTRandomDrawFactoryProxy.sol::5 => import {IVRFNFTRandomDrawFactory} from "./interfaces/IVRFNFTRandomDrawFactory.sol";
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::5 => import {Initializable} from "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
```
#### Tools used
Manual code review

### 3rd Bug
Use Shift Right/Left instead of Division/Multiplication if possible

#### Impact
Issue Information: 
A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.

While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

Example
🤦 Bad:

uint256 b = a / 2;
uint256 c = a / 4;
uint256 d = a * 8;
🚀 Good:

uint256 b = a >> 1;
uint256 c = a >> 2;
uint256 d = a << 3;

#### Findings:
```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::31 => uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
2022-12-forgeries/src/VRFNFTRandomDraw.sol::33 => uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```
#### Tools used
Manual code review
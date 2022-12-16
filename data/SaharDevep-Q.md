
### Unsafe ERC20 Operation(s)

#### Impact
Issue Information: [L001](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l001---unsafe-erc20-operations)

#### Findings:
```
examples\VRFNFTRandomDraw.sol::187 => IERC721EnumerableUpgradeable(settings.token).transferFrom(
examples\VRFNFTRandomDraw.sol::295 => IERC721EnumerableUpgradeable(settings.token).transferFrom(
examples\VRFNFTRandomDraw.sol::315 => IERC721EnumerableUpgradeable(settings.token).transferFrom(
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Unspecific Compiler Version Pragma

#### Impact
Issue Information: [L003](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l003---unspecific-compiler-version-pragma)

#### Findings:
```
examples\utils\Version.sol::2 => pragma solidity ^0.8.10;
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)


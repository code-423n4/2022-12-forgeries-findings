## Summary<a name="Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | Contracts are not using their OZ Upgradeable counterparts | 1 |

Total: 1 contexts over 1 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Use a more recent version of Solidity | 1 |
| [NC&#x2011;2](#NC&#x2011;2) | Public Functions Not Called By The Contract Should Be Declared External Instead | 1 |
| [NC&#x2011;3](#NC&#x2011;3) | Implementation contract may not be initialized | 1 |
| [NC&#x2011;4](#NC&#x2011;4) | Avoid Floating Pragmas: The Version Should Be Locked | 1 |
| [NC&#x2011;5](#NC&#x2011;5) | Use of Block.Timestamp | 4 |
| [NC&#x2011;6](#NC&#x2011;6) | Lines are too long | 1 |
| [NC&#x2011;7](#NC&#x2011;7) | Event emit should emit a parameter | 1 |

Total: 10 contexts over 7 issues

## Low Risk Issues




### <a href="#Summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Contracts are not using their OZ Upgradeable counterparts

The non-upgradeable standard version of OpenZeppelin’s library are inherited / used by the contracts.
It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

#### <ins>Proof of Concept</ins>

```solidity
4: import {ERC1967Proxy} from "@openzeppelin/contracts/proxy/ERC1967/ERC1967Proxy.sol";

```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDrawFactoryProxy.sol#L4



#### <ins>Recommended Mitigation Steps</ins>

Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.



## Non Critical Issues

### <a href="#Summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Use a more recent version of Solidity

<a href="https://blog.soliditylang.org/2021/04/21/solidity-0.8.4-release-announcement/">0.8.4</a>:
bytes.concat() instead of abi.encodePacked(<bytes>,<bytes>)

<a href="https://blog.soliditylang.org/2022/02/16/solidity-0.8.12-release-announcement/">0.8.12</a>: 
string.concat() instead of abi.encodePacked(<str>,<str>)

<a href="https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/">0.8.13</a>: 
Ability to use using for with a list of free functions

<a href="https://blog.soliditylang.org/2022/05/18/solidity-0.8.14-release-announcement/">0.8.14</a>:

ABI Encoder: When ABI-encoding values from calldata that contain nested arrays, correctly validate the nested array length against calldatasize() in all cases.
Override Checker: Allow changing data location for parameters only when overriding external functions.

<a href="https://blog.soliditylang.org/2022/06/15/solidity-0.8.15-release-announcement/">0.8.15</a>:

Code Generation: Avoid writing dirty bytes to storage when copying bytes arrays.
Yul Optimizer: Keep all memory side-effects of inline assembly blocks.

<a href="https://blog.soliditylang.org/2022/08/08/solidity-0.8.16-release-announcement/">0.8.16</a>:

Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.

<a href="https://blog.soliditylang.org/2022/09/08/solidity-0.8.17-release-announcement/">0.8.17</a>:

Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity ^0.8.10;
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/utils\Version.sol#L2



#### <ins>Recommended Mitigation Steps</ins>

Consider updating to a more recent solidity version.





### <a href="#Summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Public Functions Not Called By The Contract Should Be Declared External Instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public.

#### <ins>Proof Of Concept</ins>


```solidity
function resignOwnership() public onlyOwner {
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/ownable\OwnableUpgradeable.sol#L114







### <a href="#Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Implementation contract may not be initialized

OpenZeppelin recommends that the initializer modifier be applied to constructors. 
Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.
https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

#### <ins>Proof Of Concept</ins>


```solidity
13: constructor(uint32 version)
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/utils\Version.sol#L13





### <a href="#Summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Avoid Floating Pragmas: The Version Should Be Locked

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

#### <ins>Proof Of Concept</ins>

```solidity
Found usage of floating pragmas ^0.8.10 of Solidity in [Version.sol]
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/utils\Version.sol#L2






### <a href="#Summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Use of Block.Timestamp

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.
References: SWC ID: 116

#### <ins>Proof Of Concept</ins>


```solidity
90: if (_settings.recoverTimelock < block.timestamp + WEEK_IN_SECONDS) {
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L90

```solidity
153: request.drawTimelock > block.timestamp
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L153

```solidity
204: if (request.drawTimelock >= block.timestamp) {
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L204

```solidity
306: if (settings.recoverTimelock > block.timestamp) {
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L306



#### <ins>Recommended Mitigation Steps</ins>
Block timestamps should not be used for entropy or generating random numbers—i.e., they should not be the deciding factor (either directly or through some derivation) for winning a game or changing an important state.

Time-sensitive logic is sometimes required; e.g., for unlocking contracts (time-locking), completing an ICO after a few weeks, or enforcing expiry dates. It is sometimes recommended to use block.number and an average block time to estimate times; with a 10 second block time, 1 week equates to approximately, 60480 blocks. Thus, specifying a block number at which to change a contract state can be more secure, as miners are unable to easily manipulate the block number.



### <a href="#Summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Lines are too long

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length
Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

#### <ins>Proof Of Concept</ins>

```solidity
64: /// @notice has chosen a random number (in case random number = 0(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0))
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/interfaces\IVRFNFTRandomDraw.sol#L64





### <a href="#Summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> Event emit should emit a parameter

Some emitted events do not have any emitted parameters. It is recommended to add some parameter such as state changes or value changes when events are emitted

#### <ins>Proof Of Concept</ins>


```solidity
33: emit SetupFactory()

```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDrawFactory.sol#L33







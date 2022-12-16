# QA Report
## Finding Summary
||Issue|Instances|
|-|-|-|
|[NC-01]|Long Lines (> 120 Characters)|5|
|[NC-02]|Order of Functions Not Compliant With Solidity Docs|3|
|[NC-03]|Order of Modifiers Not Compliant With Solidity Docs|1|
|[NC-04]|NatSpec Comments Use `//` Instead of `///`|1|
|[NC-05]|Inconsistent `.` In Comments|1|
|[NC-06]|Misleading Comment / Possible Typo|1|
|[NC-07]|`Winter` - `Winner` Typo|1|

### [NC-01] Long Lines (> 120 Characters)

Lines with greater length than 120 characters are used. The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-lengthhttps://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) suggests that all lines should be 120 characters or less in width.

#### Findings:

*/src/VRFNFTRandomDrawFactory.sol*
Links: [4](https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDrawFactory.sol#L4).
```solidity
4:	import {IERC721EnumerableUpgradeable} from "@openzeppelin/contracts-upgradeable/token/ERC721/extensions/IERC721EnumerableUpgradeable.sol";
```

*/src/VRFNFTRandomDraw.sol*
Links: [5](https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L5).
```solidity
5:	import {IERC721EnumerableUpgradeable} from "@openzeppelin/contracts-upgradeable/token/ERC721/extensions/IERC721EnumerableUpgradeable.sol";
```

*/src/interfaces/IVRFNFTRandomDraw.sol*
Links: [64](https://github.com/code-423n4/2022-12-forgeries/tree/main/src/interfaces/IVRFNFTRandomDraw.sol#L64), [78](https://github.com/code-423n4/2022-12-forgeries/tree/main/src/interfaces/IVRFNFTRandomDraw.sol#L78), [82](https://github.com/code-423n4/2022-12-forgeries/tree/main/src/interfaces/IVRFNFTRandomDraw.sol#L82).
```solidity
64:	        /// @notice has chosen a random number (in case random number = 0(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0))
78:	        /// @notice Start token ID for the drawing (if totalSupply = 20 but the first token is 5 (5-25), setting this to 5 would fix the ordering)
82:	        /// @notice Draw buffer time – time until a re-drawing can occur if the selected user cannot or does not claim the NFT.
```

### [NC-02] Order of Functions Not Compliant With Solidity Docs

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) suggests the following function ordering:  constructor, receive function (if exists), fallback function (if exists), external, public, internal, private.

#### Findings:

The following contracts are not compliant (examples are only to prove the functions are out of order NOT a full description): 

[Trading.sol](https://github.com/code-423n4/2022-12-forgeries/tree/main/src/utils/Version.sol): the constructor is positioned after an external function.
[VRFNFTRandomDraw.sol](https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol): external functions are positioned after internal functions.
[OwnableUpgradeable.sol](https://github.com/code-423n4/2022-12-forgeries/tree/main/src/ownable/OwnableUpgradeable.sol): public functions are positioned after internal functions.

### [NC-03] Order of Functions Not Compliant With Solidity Docs

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#function-declaration) suggests the following modifer ordering:  Visibility, Mutability, Virtual, Override, Custom modifiers.

#### Findings:

`initialize` has a custom modifier after a visibility modifier.

*/src/VRFNFTRandomDrawFactory.sol*
Links: [31](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L31)
```solidity
31:	function initialize(address _initialOwner) initializer external {
```

### [NC-04] NatSpec Comments Use `//` Instead of `///`

The [NatSpec notation](https://docs.soliditylang.org/en/v0.8.17/natspec-format.html#documentation-example) requires the use of `///` or `/**` to differentiate from regular comments.

#### Findings:

*/src/VRFNFTRandomDraw.sol*
Links: [32](https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L32).
```solidity
32:	// @dev about 30 days in a month
```

### [NC-05] Inconsistent `.` In Comments

Most comments in the codebase do not end with a `.`. It is best to keep a consistent style.

#### Findings:

*/src/VRFNFTRandomDraw.sol*
Links: [21](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L21), [39](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L39), [142](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L142), [143](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L143), [253](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L253), [278](#L278), [281](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L281), [294](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L294).
```solidity
21:	/// @notice Our callback is just setting a few variables, 200k should be more than enough gas.
39:	/// @notice Settings used for the contract.
142:	// Chainlink request cannot be currently in flight.
143:	// Request is cleared in re-roll if conditions are correct.
253:	// We know there will only be 1 word sent at this point.
278:	// Assume (potential) winner calls this fn, cache.
281:	// Check if this user has indeed won.
294:	// Transfer token to the winter.
```

*/src/interfaces/IVRFNFTRandomDraw.sol*
Links: [76](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L76), [82](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L82).
```solidity
76:	/// @notice Token that each (sequential) ID has a entry in the raffle.
82:	/// @notice Draw buffer time – time until a re-drawing can occur if the selected user cannot or does not claim the NFT.
```

*/src/interfaces/IVRFNFTRandomDrawFactory.sol*
Links: [7](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDrawFactory.sol#L7).
```solidity
7:	/// @notice Cannot be initialized with a zero address impl.
```

*/src/ownable/OwnableUpgradeable.sol*
Links: [89](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L89), [113](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L113).
```solidity
89:	/// @dev Ensure is called only from trusted internal code, no access control checks.
113:	/// @dev only callably by the owner, dangerous call.
```

### [NC-06] Misleading Comment / Possible Typo

There is a seemingly odd comment that appears to have been copy and pasted by accident multiple times. Consider describing the comment in more detail if intentional or remove the excess if not.

#### Findings:

*/src/interfaces/IVRFNFTRandomDraw.sol*
Links: [64](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L64).
```solidity
64:	/// @notice has chosen a random number (in case random number = 0(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0))
```

### [NC-07] `Winter` - `Winner` Typo

Based on other comments and context in the `winnerClaimNFT` function it seems as though `Winner` was misspelled as `Winter` on [L294](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L294). 

#### Findings:

*/src/VRFNFTRandomDraw.sol*
Links: [294](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L294).
```solidity
294:	// Transfer token to the winter.
```
**Suggested Change**
```solidity
294:	// Transfer token to the winner.
```
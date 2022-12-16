# QA Report

## Misleading comments
There are two instances of comments which contradict the code. Either the comment or the code ought to be updated:
- `VRFNFTRandomDraw:initialize()` comment states that the size of the drawing range must be "at least two" (`>=2`), but the require statement uses a strict inequality which actually enforces "at least three" (`>2`)
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L111
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L114
- `VRFNFTRandomDraw:lastResortTimelockOwnerClaimNFT()` comment contains `"If recoverTimelock is not setup, or if not yet occurred"`, however due to `initialize()` logic it is impossible that it would not be setup
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L305
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L90-L98

<br>

## Non-ERC721 contracts (e.g. CryptoPunks) incompatible with draw contract
NFTs that do not adhere to the ERC721 standard will not be usable in `VRFNFTRandomDraw`, as the `IERC721EnumerableUpgradeable` interface is used when interacting with the NFT contract. While non-ERC721 NFTs are rare, it is worth noting since CryptoPunks were mentioned as an example drawing NFT in the contest overview/README.

<br>

## Use solidity built in time constants
Instead of manually calculating seconds, use the build in keywords `minutes`, `hours` and `days`. This reduces margin for error and improves readability (the calculation for `MONTH_IN_SECONDS` is actually incorrect by a factor of 7).
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L29
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L31
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L33

<br>

## No enforcement for strictly increasing versions
`Version` is intended to keep track of implementation versions. However it is possible to upgrade to a new implementation with a version that is less than or equal to the old version. Strictly increasing versions should be enforced for each proxy.

<br>

## Use fixed compiler version rather than floating one
A floating pragma risks a different compiler version being used in production vs testing, which poses security risks.
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol#L2

<br>

## Use `constant` instead of `immutable`
For immutable variables whose value is set outside of the constructor, the `constant` keyword ought to be used instead to save gas and improve readability.
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L22
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L24
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L26
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L29
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L31
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L33

<br>

### Use `safeTransferFrom` instead of `transferFrom`
Concerning ERC721 tokens, `transferFrom` can result in tokens being lost if they are sent to a contract that does not implement `IERC721Receiver.onERC721Received`. OpenZeppelin encourages the use of `safeTransferFrom` over `transferFrom` for this reason.
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L187
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L295
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L315

<br>

### Mark state variables as either `public` or `private`
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L22
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L24
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L26
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L29
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L31
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L33
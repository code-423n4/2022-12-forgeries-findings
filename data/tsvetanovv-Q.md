## [QA-01] Use latest Solidity version

```
Version.sol and other contracts
```

Solidity pragma versioning should be upgraded to latest available version. Currently the solidity version in contracts is ^0.8.10 which was found to possess some bugs.
[Link](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol#L2)

## [QA-02] Use stable pragma statement

```
Version.sol
```

Using a floating pragma `^` statement is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode.
[Link](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol#L2)

## [QA-03] Different pragma directives are used

#### Recommendation

Use one Solidity version on each contract.

## [QA-04] EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING

#### Code contains empty block

```2 results - 2 files
src/VRFNFTRandomDrawFactory.sol:
55:     function _authorizeUpgrade(address newImplementation)
56:            internal
57:            override
58:            onlyOwner
59:        {}
```

#### Recommended Mitigation Steps

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

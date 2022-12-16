# QA Report for Forgeries contest
## Overview
During the audit, 1 low and 5 non-critical issues were found.

№ | Title | Risk Rating  | Instance Count
--- | --- | --- | ---
L-1 | Incomplete comment | Low | 1
NC-1 | Order of Functions | Non-Critical | 7
NC-2 | Typos | Non-Critical | 2
NC-3 | Missing NatSpec | Non-Critical | 1
NC-4 | Empty function bodies | Non-Critical | 1
NC-5 | Maximum line length exceeded | Non-Critical | 2

## Low Risk Findings(1)
### L-1. Incomplete comment
##### Instances
- [```// If the number has been drawn and```](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L148)

#
## Non-Critical Risk Findings(5)
### NC-1. Order of Functions
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-functions), ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.  
Functions should be grouped according to their visibility and ordered:
1) constructor
2) receive function (if exists)
3) fallback function (if exists)
4) external
5) public
6) internal
7) private
##### Instances
```initialize()``` function is not right after constructor:
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L72-L138

Constructor after function:
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol#L13

internal functions before external/public:
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L140-L169
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L227-L259

internal function before external public:
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L55-L65

external functions after public:
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L302-L300
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L276-L321

##### Recommendation
Reorder functions where possible.
#
### NC-2. Typos
##### Instances
- [```// Transfer token to the winter.```](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L294) => ```minter```
- [```/// @notice When the owner reclaims nft aftr recovery time delay```](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L46) => ```after```

#
### NC-3. Missing NatSpec
##### Instances
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L31

##### Recommendation
Add NatSpec for all functions.
#
### NC-4. Empty function bodies
##### Instances
- https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L59

#
### NC-5. Maximum line length exceeded
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#maximum-line-length), maximum suggested line length is 120 characters.
##### Instances
- (282 symbols) https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L64
- (138 symbols) https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L78

##### Recommendation
Make the lines shorter.

## Hardcoded version
### Summary
Version contract is called once in the code with a hardcoded value of 1 multiple times
### Details
Starting value of version is 1, what is kind of logic from everybody perspective. This value is hardcoded in the upgradeable contracts of the code.

However, code doesn't consider 0 as a starting value, neither check it, therefore in the future a version 0 can be created what would be confusing.

### Github Permalinks
https://github.com/code-423n4/2022-12-forgeries/blob/1e689ae80da66027d32c75becc072abe946dec7a/src/VRFNFTRandomDrawFactory.sol#L18
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L19

### Mitigation
This version value can be stored as a constant for more visibility of the users and code maintainers and called in the constructor.

Also consider if the starting value can be 0, and if not block it. 



## Different versions of pragma
### Summary
Some of the contracts include an unlocked pragma, e.g., pragma solidity >=0.8.10. 

Locking the pragma helps ensure that contracts are not accidentally deployed using an old compiler version with unfixed bugs.

### Github Permalinks
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/utils/Version.sol#L2
    pragma solidity ^0.8.10;
### Mitigation
Lock pragmas to a specific Solidity version. 
Consider converting ^0.8.10 into 0.8.16



## Bad assumption of dates
### Summary
MONTH_IN_SECONDS is being calculated taking 30 days as days of month, however, following that operation 12 * 30 would be 360. However, days are 365 (or 365 in gap years). Later this wrongly operation is performed in a check of time
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L93-L96
```
    /// @dev 60 seconds in a min, 60 mins in an hour
    uint256 immutable HOUR_IN_SECONDS = 60 * 60; 
    /// @dev 24 hours in a day 7 days in a week
    uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
    // @dev about 30 days in a month
    uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;

```
```
        if (
            _settings.recoverTimelock >
            block.timestamp + (MONTH_IN_SECONDS * 12)
        ) {
```
### Mitigation
Consider using real values and not approximations that can lead into wrong assumptions

## Bad order of code
### Summary
Clearness of the code is important for the readability and maintainability.
As Solidity guidelines says about declaration order:
1.Type declarations
2.State variables
3.Events
4.Modifiers
5.Functions
Also, state variables order affects to gas in the same way as ordering structs for saving storage slots

### Github Permalink
- event declared after functions
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/interfaces/IVRFNFTRandomDrawFactory.sol#L18

### Mitigation
Follow solidity style guidelines https://docs.soliditylang.org/en/v0.8.15/style-guide.html




## Missing Natspec 
### Summary 
Missing Natspec and regular comments affect readability and maintainability of a codebase. 

### Details 
Contracts has partial or full lack of comments

### Github Permalinks 
- Natspec `@param`
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/utils/Version.sol#L13-L15
 ### Mitigation
 - Add `@param` descriptors


## Maximum line length exceeded
### Summary
Long lines should be wrapped to conform with Solidity Style guidelines. 
### Details 
Lines that exceed the 120 character length suggested by the Solidity Style guidelines. Reference: https://docs.soliditylang.org/en/v0.8.16/style-guide.html#maximum-line-length
### Github Permalinks 
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDrawFactory.sol#L4

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L5

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L250

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/interfaces/IVRFNFTRandomDraw.sol#L64

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/interfaces/IVRFNFTRandomDraw.sol#L78

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/interfaces/IVRFNFTRandomDraw.sol#L82


### Mitigation
Reduce line length to less than 120 at least to improve maintainability and readability of the code 



## Immutable variables assigned without a constructor should be constant and written in UPPERCASE
### Summary
`constant` keyword helps with readability of the code and to make sure that they do not change. 

### Details
Immutable variables that are assigned without a constructor/initializer can be constant

### Github Permalinks

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L21-L34


### Mitigation
Add constant and change `VariableName` to `VARIABLE_NAME`






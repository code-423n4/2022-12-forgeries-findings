## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol

```solidity
// function coming before constructor
9:  function contractVersion() external view returns (uint32) {
```

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDrawFactory.sol

```solidity
// events should come before all functions
18:    event SetupFactory();
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), public, external, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol

```solidity

// these external views should come after all external non-view functions
58:    function getRequestDetails()
264:    function hasUserWon(address user) public view returns (bool) {


// these public view are coming after external ones
75:    function initialize(address admin, Settings memory _settings)
264:   function hasUserWon(address user) public view returns (bool) {

// this internal function is coming before external and public ones
141:    function _requestRoll() internal {
```

---

### natSpec missing [3] 

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol

```solidity

// incomplete
24:    constructor(address _implementation) initializer {

31:    function initialize(address _initialOwner) initializer external {

// incomplete, missing @returns
38:    function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)

```

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol

```solidity
4:  contract Version {

13:  constructor(uint32 version) {
```

---

### Version [4]

- Pragma versions should be standardized and avoid floating pragma `( ^ )`

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol

```solidity
// this version does not match with the others
2:  pragma solidity ^0.8.10;
```

---


## G-01 USAGE OF UINTS OR INTS SMALLER THAN 32 BYTES INCURS OVERHEAD

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

[https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html) Use a larger size then downcast where needed

_There are 4 instances of this issue_

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol

```
File: src/VRFNFTRandomDraw.sol

24: uint16 immutable minimumRequestConfirmations = 3;
26: uint16 immutable wordsRequested = 1;
```

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol

```
File: src/utils/Version.sol

5: uint32 private immutable __version;
13: constructor(uint32 version) {
```

------

## G-02 USING BOOLS FOR STORAGE INCURS OVERHEAD

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot's contents, replace the bits taken up by the boolean, and then write back. This is the compiler's defense against contract upgrades and pointer aliasing, and it cannot be disabled.

_There are 3 instances of this issue_

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol

```
File: src/VRFNFTRandomDraw.sol

63: bool hasChosenRandomNumber,
```

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol

```
File: src/interfaces/IVRFNFTRandomDraw.sol

65: bool hasChosenRandomNumber;
126: bool hasChosenRandomNumber,
```

-------


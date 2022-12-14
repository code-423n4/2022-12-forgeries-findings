| | issue |
| ----------- | ----------- 
| 1 | [Structs can be packed into fewer storage slots](#1-structs-can-be-packed-into-fewer-storage-slots) |
| 2 | [state variables should be cached in stack variables rather than re-reading them from storage](#2-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage) |
| 3 | [usage of uint/int smaller than 32 bytes (256 bits) incurs overhead](#3-usage-of-uintint-smaller-than-32-bytes-256-bits-incurs-overhead) |
| 4 | [public functions not called by the contract should be declared external instead](#4-public-functions-not-called-by-the-contract-should-be-declared-external-instead) |
| 5 | [precalculate operation instead](#5-precalculate-operation-instead) |

## 1. Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings

should put `subscriptionId` near one of `token` or `drawingToken` so they go inside one slot 
- [IVRFNFTRandomDraw.sol#L71-L90](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L71-L90)


## 2. state variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

`request.drawTimelock` should be cached before the if statement because its used again after
- [VRFNFTRandomDraw.sol#L152](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L152)

`settings.drawingTokenStartId` cache before 249
- [VRFNFTRandomDraw.sol#L250](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L250)

`settings.token` cache before 287
- [VRFNFTRandomDraw.sol#L289](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L289)

`settings.tokenId` cache before 287
- [VRFNFTRandomDraw.sol#L290](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L290)


## 3. usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Use a larger size then downcast where needed

- [Version.sol#L5](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol#L5)


## 4. public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

`owner`
- [OwnableUpgradeable.sol#L68](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L68)

`pendingOwner`
- [OwnableUpgradeable.sol#L73](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L73)

`transferOwnership`
- [OwnableUpgradeable.sol#L79](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L79)

`safeTransferOwnership`
- [OwnableUpgradeable.sol#L102](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L102)

`resignOwnership`
- [OwnableUpgradeable.sol#L114](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L114)

`acceptOwnership`
- [OwnableUpgradeable.sol#L119](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L119)

`cancelOwnershipTransfer`
- [OwnableUpgradeable.sol#L128](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L128)


## 5. precalculate operation instead

pre calculate `MONTH_IN_SECONDS * 12` and use that value instead of using a operation 

- [VRFNFTRandomDraw.sol#L95](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L95)
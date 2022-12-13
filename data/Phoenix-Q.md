## Invoke the `_disableInitializers` function in the constructor to automatically lock it during deployment

`File: src/VRFNFTRandomDraw.sol`

```solidity
constructor() {
    _disableInitializers();
 }

```

## Try using uint instead of bools for storage to avoid overhead

[`IVRFNFTRandomDraw.sol#L65`](https://github.com/code-423n4/2022-12-forgeries/blob/82a5fc7540249325ae48498720d40d943ee17e27/src/interfaces/IVRFNFTRandomDraw.sol#L65)

```solidity
   struct CurrentRequest {
        ...
        bool hasChosenRandomNumber;
        ...
    }
```

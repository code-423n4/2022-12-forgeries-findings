## Invoke the `_disableInitializers` function in the constructor to automatically lock it during deployment

`File: src/VRFNFTRandomDraw.sol`

```solidity
constructor() {
    _disableInitializers();
 }

```
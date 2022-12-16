# Gas Report
## Finding Summary
||Issue|Instances|
|-|-|-|
|[G-01]|`uint<size>` Used Rather Than `uint256`|4|
|[G-02]|`Settings` Struct Can Be Packed Into Fewer Slots|1|

### [G-01] `uint<size>` Used Rather Than `uint256`

The EVM deals with 32 bytes at a time. For `uint<size>`s less than 32 bytes the EVM performs more operations as a result of the needed size reduction from `uint256` to `uint<size>`. Consider using `uint256`.

#### Findings:

*/src/utils/Version.sol*
Links: [5](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol#L5).
```solidity
5:	uint32 private immutable __version;
```

*/src/VRFNFTRandomDraw.sol*
Links: [22](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L22), [24](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L24), [26](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L26).
```solidity
22:	uint32 immutable callbackGasLimit = 200_000;
24:	uint16 immutable minimumRequestConfirmations = 3;
26:	uint16 immutable wordsRequested = 1;
```

### [G-02] `Settings` Struct Can Be Packed Into Fewer Slots

The `Settings` struct on [L71](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L71) of [IVRFNFTRandomDraw.sol](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol) uses 9 32 byte slots: [[20], [32], [20], [32], [32], [32], [32], [32], [8]]. The last variable `uint64` can be moved to the start to allow for 8 slots (thus saving gas): [[8, 20], [20], [32], [32], [32], [32], [32], [32]].
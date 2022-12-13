# Report


## Gas Optimizations

### 1 . Using Unpacked Structs costs more gas 

Code :

```solidity
File: src/VRFNFTRandomDraw.sol

80:          settings = _settings;
       
```
[Link](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L80)

The Settings struct in `src/interfaces/IVRFNFTRandomDraw.sol` is not packed correctly .

If we look into it .

```solidity

    struct Settings {
        /// @notice Token Contract to put up for raffle
        address token;
        /// @notice Token ID to put up for raffle
        uint256 tokenId;
        /// @notice Token that each (sequential) ID has a entry in the raffle.
        address drawingToken;
        /// @notice Start token ID for the drawing (if totalSupply = 20 but the first token is 5 (5-25), setting this to 5 would fix the ordering)
        uint256 drawingTokenStartId;
        /// @notice End token ID for the drawing (exclusive) (token ids 0 - 9 would be 10 in this field)
        uint256 drawingTokenEndId;
        /// @notice Draw buffer time – time until a re-drawing can occur if the selected user cannot or does not claim the NFT.
        uint256 drawBufferTime;
        /// @notice block.timestamp that the admin can recover the NFT (as a safety fallback)
        uint256 recoverTimelock;
        /// @notice Chainlink gas keyhash
        bytes32 keyHash;
        /// @notice Chainlink subscription id
        uint64 subscriptionId;
    }
```
**The storage slots be like**

```
slot 1
address token - 20

slot 2
uint256 toeknId - 32

slot 3 
address drawingToken - 20

slot 4 
uint256 drawingTokenStartId - 32

slot 5
uint256 drawBufferTime - 32

slot 6 
uint256 recoverTimelock - 32

slot 7
bytes32 keyHash - 32

slot 8
uint64 subcriptionId - 8

```

We can Optimize this by packing the same datatypes in order .

**The packed version :**

```
slot 1
uint256 tokenId - 32

slot 2
uint256 drawingTokenStartId - 32

slot 3
uint256 drawingTokenEndId - 32

slot 4
uint256 drawBufferTime - 32

slot5
uint256 recoverTimelock - 32

slot 6
bytes32 keyHash - 32

slot 7
address token - 20
uint64 subcriptionId - 8

```
In this way we can save one storage slot (20k) gas  

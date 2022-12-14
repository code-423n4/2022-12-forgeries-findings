## Check currentChainlinkRequestId twice
When call VRFNFTRandomDraw.startDraw it does check request.currentChainlinkRequestId then inside _requestRoll(), it check again.
### Proof of Concept
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L175
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L144
### Recommended Mitigation Steps
Remove the check in _requestRoll

## Add unchecked {} for subtractions where the operands cannot underflow because of a previous
When initialize, it already check _settings.drawingTokenEndId > _settings.drawingTokenStartId, then when we calculate tokenRange in fulfillRandomWords, it can be added unchecked
### Proof of Concept
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L249

## Redraw delete request then update it again in _requestRoll
When call VRFNFTRandomDraw.redraw, it will delete the request state, then call to _requestRoll which will update the drawTimelock and currentChainlinkRequestId.
It will cost gas to update state from non-zero to zero, then assign them into non-zero again
### Proof of Concept
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L209
### Recommended Mitigation Steps
We can set request.currentChosenTokenId and request.hasChosenRandomNumber to zero and false instead of delete the request state.
Then when it call _requestRoll, the value request.currentChainlinkRequestId and request.drawTimelock will be update


## STRUCTS CAN BE PACKED INTO FEWER STORAGE SLOTS
Struct Settings can be packed to save gas
### Proof of Concept
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L71-L90
### Recommended Mitigation Steps
```solidity
struct Settings {
    /// @notice Token ID to put up for raffle
    uint256 tokenId;
    /// @notice Start token ID for the drawing (if totalSupply = 20 but the first token is 5 (5-25), setting this to 5 would fix the ordering)
    uint256 drawingTokenStartId;
    /// @notice End token ID for the drawing (exclusive) (token ids 0 - 9 would be 10 in this field)
    uint256 drawingTokenEndId;
    /// @notice Draw buffer time – time until a re-drawing can occur if the selected user cannot or does not claim the NFT.
    uint256 drawBufferTime;
    /// @notice block.timestamp that the admin can recover the NFT (as a safety fallback)
    uint256 recoverTimelock;
    /// @notice Token that each (sequential) ID has a entry in the raffle.
    address drawingToken;
    /// @notice Token Contract to put up for raffle
    address token;
    /// @notice Chainlink gas keyhash
    bytes32 keyHash;
    /// @notice Chainlink subscription id
    uint64 subscriptionId;
}
```

1. `redraw()` can be called before `startDraw()`.
    
    Whenever `startDraw()` has not yet been called, `redraw()` could be called to initiate the first draw. However, this bypasses the `SetupDraw` event emit, which might confuse users. 
    
    [https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L173-L225](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L173-L225)
    
2. It’s recommended to use Solidity time units (`minutes`, `hours`, `days`) to improve readability.
    
    [https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L29-L33](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L29-L33)
    
3. `redraw()` doesn’t emit an event.
    
    `redraw()` is a fairly important functionality which benefits from an emit event similarly to `SetupDraw` in `startDraw()`
    
    [https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L203-L225](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L203-L225)
    
4. Missing verification in `fulfillRandomWords()` to check whether the winning tokenId has an owner.
    
    In case the winning tokenId of the drawingToken has no owner (e.g. is not yet minted/doesn’t exist), the contract has to wait for `drawBufferTime` before it can be rerolled. If the winning tokenId has no owner, we can redraw directly and the contract is not ‘stuck’ for a determined time.
    
    [https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L230-L260](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L230-L260)
    
5. The contract does not support the approvers of the winning tokenId to withdraw the raffle tokenId. 
    
    In some cases tokens are managed by the approvers (e.g. particular wallets/contracts). In case this particular NFT wins, the approvers will not be able to withdraw the raffle token id as `winnerClaimNFT()` checks whether `msg.sender` is the owner of the NFT.
    
    [https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L264-L300](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L264-L300)
    
6. The initializer of `VRFNFTRandomDrawFactory` does not call `__UUPSUpgradeable_init()`  
    
    [https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L31-L34](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L31-L34)
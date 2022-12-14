G1. https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L149-L156
These checks can be deleted to save gas since in Line 144, we have checked ``request.currentChainlinkRequestId != 0``, so if ``request.currentChainlinkRequestId == 0`` holds, then it means a new request is made, and there is no need to check other fields.  

G2. https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L175-L177
Checking ``request.currentChainlinkRequestId != 0`` can be eliminated here since the following call ``_requestRoll()`` will check this condition anway in its body. There is no need to check the same condition twice. 

G3. https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L58
Caching state variable ``request`` can save gas
```
function getRequestDetails()
        external
        view
        returns (
            uint256 currentChainlinkRequestId,
            bool hasChosenRandomNumber,
            uint256 drawTimelock
        )
    {
       IVRFNFTRandomDraw.CurrentRequest memory req = request;

 
        currentChainlinkRequestId = req.currentChainlinkRequestId;
        hasChosenRandomNumber = req.hasChosenRandomNumber;
        drawTimelock = req.drawTimelock;
    }
```

G4: https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L80
Moving this line (effects) to the last line of the ``initialize`` function can save gas. 

G5: https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L65
Using 1 to encode false and 2 to encode true will save gas since changing from zero to non-zero is more expensive

G6. https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L126-L137
Simplifying the try-catch stmt can save gas; see below:
```
if(IERC721EnumerableUpgradeable(_settings.token).ownerOf(_settings.tokenId) != admin) revert DOES_NOT_OWN_NFT();
```
G7. https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L26
Changing it to private can save gas

G8 https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L112-L117
Eliminating the first condition can save gas since the second condition + the underflow check by safe math introduced by solidity > 0.8.0 is sufficient to cover condition 1
```
if ( _settings.drawingTokenEndId - _settings.drawingTokenStartId < 2 ) 
            revert DRAWING_TOKEN_RANGE_INVALID();

```

G9. https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L230-L233
Caching ``setting`` can save gas for this function. 
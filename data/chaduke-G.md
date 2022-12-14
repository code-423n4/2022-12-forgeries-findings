G1. https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L149-L156
These checks can be deleted to save gas since in Line 144, we have checked ``request.currentChainlinkRequestId != 0``, so if ``request.currentChainlinkRequestId == 0`` holds, then it means a new request is made, and there is no need to check other fields.  

G2. https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L175-L177
Checking ``request.currentChainlinkRequestId != 0`` can be eliminated here since the following call ``_requestRoll()`` will check this condition anway in its body. There is no need to check the same condition twice. 


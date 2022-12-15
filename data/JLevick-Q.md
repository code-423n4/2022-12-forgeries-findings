### [L01] Incorrect MONTH_IN_SECONDS.

**Description:**
The variable MONTH_IN_SECONDS is calculated incorrectly. Is is multiplied by 7 and then 30 which will result in its value being the equivalent of 30 weeks not 1 month. As a result settings.drawBufferTime & settings.recoverTimelock may be able to be set to values that should be restricted.

**Location:**
[VRFNFTRandomDraw.sol#L33](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L33) 

**Recommendation:**
Change line 33 to: `uint256 immutable MONTH_IN_SECONDS = (3600 * 24) * 30;` 
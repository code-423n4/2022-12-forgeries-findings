### [G01] Caching State Variables that are used multiple times.

**Description:**
Whenever a state variable is used more than once in a given scope without being modified you can save gas by caching the variable and using that value instead.

**Instances:**
[VRFNFTRandomDraw.sol#L249-L256](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L249-L256) - Can cache settings.drawingTokenStartId.
[VRFNFTRandomDraw.sol#L289-L298](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L289-L298) - Can cache settings.token & settings.tokenId.

**Gas Savings:**
| VRFNFTRandomDraw | Pre Changes | Post Changes | Savings | 
| --- |--- |--- |--- |
| Deployment | 1,237,526 | 1,236,926 | 600 |
| rawFulfillRandomWords | 38,496 | 38,415 | 81 |
| winnerClaimNFT | 11,638 | 11,576 | 62 |


### [G02] Order of Error Checks.

**Description:**
When there are multple error checks in a row, it is best to place the ones involving state variables at the end to prevent unnecessary SLOAD's in instances where the error checks fail.

**Instances:**
[VRFNFTRandomDraw.sol#L235-L243](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L235-L243) - Recommend switching the if statements around.

**Gas Savings:**
~100 gas (1 SLOAD) would be saved for everytime the error is triggered on [line 241-243.](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L241-L243) 

### [G03] Math that can be Unchecked.

**Description:**
Mathematical calculations that can never over/underflow can be wrapped in unchecked blocks to save gas.

**Instances:**
[VRFNFTRandomDraw.sol#L159](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L159) - settings.drawBufferTime can never be higher than MONTH_IN_SECONDS (is currently 18,144,000 but I have mentioned in my QA it should be 2,592,000) & when added to block.timestamp will never realistically overflow.
[VRFNFTRandomDraw.sol#L248-L256](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L248-L256) - drawingTokenStartId can never be > than drawingTokenEndId due to the require check in initialise.

**Gas Savings:**
| VRFNFTRandomDraw | Pre Changes | Post Changes | Savings | 
| --- |--- |--- |--- |
| Deployment | 1,237,526 | 1,229,519 | 8,007 |
| startDraw | 86,573 | 86,511 | 62 |
| redraw | 28,944 | 28,917 | 27 | 

1. Constant definition problem

the `MONTH_IN_SECONDS` should be `3600 * 24 * 30` instead of `(3600 * 24 * 7) * 30`

2. Missing event record

the owner role can redraw NFTs by calling the redraw function. But the lack of event logging here is not conducive to user and community review.
Code Location:
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L203-L225

Solution: Add the corresponding event

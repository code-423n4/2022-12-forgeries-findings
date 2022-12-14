## N. Missing checks for address(0) when assigning values to address state variables
VRFNFTRandomDraw Line 51: coordinator = _coordinator;
### Proof of Concept
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L51
### Recommended Mitigation Steps
Add zero check

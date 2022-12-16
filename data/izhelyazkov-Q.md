Owner can renounce ownership.
The owner of `VRFNFTRandomDraw` contract can perform some privileged activities.
Since he inherits from Openzeppelin's implementation of OwnableUpgradeable, 
he can call `renounceOwnership` at any time to renounce his owner rights, 
leaving the following functions in `VRFNFTRandomDraw.sol` uncallable:
`function startDraw() external onlyOwner returns (uint256)` - line 173
`function redraw() external onlyOwner returns (uint256)` - line 203
`function lastResortTimelockOwnerClaimNFT() external onlyOwner` - line 304
Recommended Mitigation Steps

Either reimplement the function to disable `renounceOwnership` or clearly specify if it is part of the contract design.
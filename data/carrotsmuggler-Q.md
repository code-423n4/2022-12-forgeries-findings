# 1. VRFNFTRandomDraw: Typo in comment
VRFNFTRandomDraw:294 ~~winter~~ winner

# 2. Consider returning `request.currentChosenTokenId` in function `getRequestDetails()`
It returns a lot of details but not the final winning tokenID, which must be read from the public struct. Consider adding the winning tokenId here for ease of use.


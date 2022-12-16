### fulfillRandomWords can revert
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L230

This looks fine to me, but goes against the [best practices of VRFs](https://docs.chain.link/vrf/v2/security/)
"If your fulfillRandomWords() implementation reverts, the VRF service will not attempt to call it a second time", which poses a permanent DOS risk.


### poorly formed comment
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L64



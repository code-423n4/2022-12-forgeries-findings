The vulnerability can be found from #L277-#L300 - winnerClaimNFT() function in VRFNFTRandomDraw.sol
It seems that the function did't check the user has already claimed their NFT or not.

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L277-#L300

To solve the issue, Developer could implement require function to check if the user has already claimed their NFT. 

Example:
```
// For tracking claimed NFT
mapping (address => bool) claimed;

// Check if the user has already claimed the NFT
require(!claimed[user], HAS_ALREADY_CLAIMED());
```


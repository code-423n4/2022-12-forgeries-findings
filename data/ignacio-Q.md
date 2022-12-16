# The initialize function that initializes important contract state can be called by anyone.
initialize() function can be called anybody when the contract is not initialized. 
Occurences:
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L75

The attacker can initialize the contract before the legitimate deployer, hoping that the victim continues to use the same contract. In the best case for the victim, they notice it and have to redeploy their contract costing gas.

Recommend using the constructor to initialize non-proxied contracts. For initializing proxy contracts, recommend deploying contracts using a factory contract that immediately calls initialize after deployment, or make sure to call it immediately after deployment and verify the transaction
## L-01 - Check of address not being a contract is not reliable

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L100-L108

We attempt to verify that the `token` and `drawingToken` used are contracts by checking the code size. This is not a reliable way as they can be bypassed by a few ways,

1. Contract that is still in the initializing state does not have code size, i.e. called in `constructor()`.
2. CREATE2 opcode can be used to pre-calculate the address of a contract before it is deployed. Contract would not have code size too in this case.

## Recommended

There is currently no way to reliably check that a given address is an EOA or a smart contract. I recommend that this line of codes are removed as it may give developers the false assumption that we are working with only EOAs, which can introduce future bugs.

## R-01 - Internal functions should be named with the _ prefix.

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L100-L108
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L230

It is recommended that we follow the naming convention by solidity. Internal functions should be prefixed with `_`. This improves readability significantly. As the rest of the contracts does follow this convention, I believe these two cases are an oversight. 

## R-02 - Use of try-catch all is not recommended

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L186-L198
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L126-L138

We are using try-catch all errors but this is not recommended as a call to `transferFrom` or `ownerOf` can fail for all sorts of reason, but we always revert with the same kind of error. This can make it difficult to figure out the exact issue when a call fails.

## Recommended
I recommend using the if-revert pattern instead to do the revert. This way, we are able to see the revert errors that are returned by the calls to other contracts.
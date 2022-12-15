https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L126

This check is redundant. The admin/owner does not need to own the NFT until the roll is being requested. The owner of the NFT up until that point is not relevant, and even if this sanity check passes, it does not say anything about the fact that the owner/admin will keep owning or holding the NFT until the roll is requested, or will ever request a roll. As a recovery mechanism is present, you can as well simplify the code and transfer the NFT into the contract immediately upon initialization.
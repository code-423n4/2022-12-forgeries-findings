## Low Risk Issues

|     | Issue                          | Instances |
| --- |:------------------------------ |:---------:|
| L-1 | Raffle state should be updated |     4      |
 
### [L-1] Raffle state should be updated

Functions where raffle state should be updated:

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L203

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L230

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L277

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L304

Raffle state is not updated when (1) The winner has claimed the NFT; (2) The `admin` has reclaimed NFT; (3) raffle just started; (4) Redraw has started. This will confuse users.

The protocol should set a raffle state to indicate the current stage of the raffle, e.g. `started, restarted, ongoing, completed, winnerHasClaimed, adminHasReclaimed, etc.` so that users can query the raffle state and avoid unnessary calling functions like `winnerClaimNFT()`.

In some cases, e.g. when the admin has reclaimed NFT, the existing raffle state is thus meaningless, but the winner,  who can no longer claim the NFT, can still query its winning state. This is meaningless and can only confuse users.
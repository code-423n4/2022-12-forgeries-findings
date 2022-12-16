1. In function `lastResortTimelockOwnerClaimNFT()` and `winnerClaimNFT()` in `VRFNFTRandomDraw.sol`  emmit event after state change.
   **summary**:  Emitting an event after a state change ensures that the event is only emitted if the state change is successful, and that the state is not modified if an exception is thrown during the function's execution. This can be useful for synchronization and state update purposes.
   **Mitigation** It is generally a good practice to emit events after the actual state change has been made in a Solidity function. In the case of the `lastResortTimelockOwnerClaimNFT` function, the `transferFrom` function call is the state change that transfers the NFT from the contract to the owner. Therefore, it is correct to have the `emit OwnerReclaimedNFT(owner())` line after the `transferFrom` function call.
2. Inconsistency in emiting events in contract `OwnableUpgradeable.sol` in functions somtimes the event are emiting before state change and sometimes after event changes.
3. In `Version.sol` the solidity version is not same as used in other contract.
   **Mitigation**: set solidity version as `0.8.16`
4. Use of `Block.timestamp`
   Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.
```solidity
src/VRFNFTRandomDraw.sol:90:        if (_settings.recoverTimelock < block.timestamp + WEEK_IN_SECONDS) {
src/VRFNFTRandomDraw.sol:95:            block.timestamp + (MONTH_IN_SECONDS * 12)
src/VRFNFTRandomDraw.sol:153:            request.drawTimelock > block.timestamp
src/VRFNFTRandomDraw.sol:159:        request.drawTimelock = block.timestamp + settings.drawBufferTime;
src/VRFNFTRandomDraw.sol:204:        if (request.drawTimelock >= block.timestamp) {
src/VRFNFTRandomDraw.sol:306:        if (settings.recoverTimelock > block.timestamp) {

```

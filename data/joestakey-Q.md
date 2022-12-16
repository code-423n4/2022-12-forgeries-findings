## Summary
### Low Risk
|      | Issue                                                                                 |
|------|---------------------------------------------------------------------------------------|
| L-01 | `winnerClaimNFT()` should handle the case when owner claimed back their NFT        |
| L-02 | `redraw()` should not have access control, as it breaks a property of hyperstructures |


## Low
## [L‑01] `winnerClaimNFT()` should handle the case when owner claimed back their NFT

The admin can recover the NFT once `recoverTimelock` has passed. But recovering with `lastResortTimelockOwnerClaimNFT` does not update `request`.

If there is a winner that then tries calling `winnerClaimNFT()`, the call will revert [here](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L295), as the `NFT` will not be owned by the raffle contract anymore. There is no custom error,  which may be confusing for the winner calling the function.

### Recommendation

Amend `winnerClaimNFT()` so that it reverts with an OWNER_WITHDREW_NFT error if the owner has claimed back their NFT.

```diff
276: /// @notice Function for the winner to call to retrieve their NFT
277:     function winnerClaimNFT() external {
278:         // Assume (potential) winner calls this fn, cache.
279:         address user = msg.sender;
280: 
281:         // Check if this user has indeed won.
282:         if (!hasUserWon(user)) {
283:             revert USER_HAS_NOT_WON();
284:         }
+            if (
+               IERC721EnumerableUpgradeable(settings.token).ownerOf(
+                   settings.tokenId
+               ) != address(this)
+            ) {
+               revert OWNER_WITHDREW_NFT();
+            }
```

## [L‑02] `redraw()` should not have access control, as it breaks a property of hyperstructures  

The contract follows the hyperstructure concept. 

One of the key characteristics of a hyperstructure is that it can continuously function without a maintainer or operator.

Quoting https://jacob.energy/hyperstructures.html, the reference given in the Readme:

```
For example, if the Uniswap team and website disappeared today the protocol will run in perpetuity
```

This is not the case here: if the owner 'disappears', there cannot be a raffle re-draw, and the raffled NFT will be stuck forever in the contract.

This breaks the 'unstopabble' property of hyperstructures

### Recommendation

Remove the `onlyOwner` modifier from `redraw()`.


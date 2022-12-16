## Summary

Overall the code was found to be well implemented and well structured. As detailed in the C4audit output and submitted gas optimization report, there are certain parts of the code which could be optimized to result in lower gas fees for the users. 

## Non critical issues

### hasUserWon should revert when the prize NFT is not owned by the contract

The view function [`hasUserWon`](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L264) should perform an additional check to ensure that the prize NFT is still owned by the contract. 
Not checking this could lead to the following situation:

* owner has withdrawn the NFT after 1 week
* winner decides to sell the token ID which has the right to the prize - they try to sell for a premium due to the second NFT being awarded to the new owner
* the winner proves that they won by having the buyer call `hasUserWon`
* the buyer buys the NFT for a premium and they are not able to withdraw the prize as this was removed by the owner

Two solutions are proposed:

1. set `request.hasChosenRandomNumber = false` when the owner calls `lastResortTimelockOwnerClaimNFT` so that the `hasUserWon` would revert.
2. add a the following within `hasUserWon`:

```javascript
// Owner of token to raffle needs to be this contract
if (
    IERC721EnumerableUpgradeable(settings.token).ownerOf(
        settings.tokenId
    ) != address(this)
) {
    revert DOES_NOT_OWN_NFT();
}
```

Implementing this would also allow the `winnerClaimNFT` function to revert earlier in the case that the winner tries to claim their prize after the owner took the prize NFT back. 
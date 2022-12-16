1:
OwnableUpgradeable.sol add Storage Gaps , It allows us to freely add new state variables in the future

Storage Gaps: https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps

```solidity
abstract contract OwnableUpgradeable is IOwnableUpgradeable, Initializable {
...

+   uint256[48] private __gap;
}
```



2:

winnerClaimNFT() Only the owner of drawingToken's NFT_ID has permission to claim, it is recommended that NFT_ID has approve can do the same, to avoid having to retrieve the drawingToken NFT first

```solidity
    function hasUserWon(address user) public view returns (bool) {
        if (!request.hasChosenRandomNumber) {
            revert NEEDS_TO_HAVE_CHOSEN_A_NUMBER();
        }

-       return
-           user ==
-           IERC721EnumerableUpgradeable(settings.drawingToken).ownerOf(
-               request.currentChosenTokenId
-           );

+           address owner = IERC721EnumerableUpgradeable(settings.drawingToken).ownerOf(request.currentChosenTokenId);
+           return (user == owner 
+                      || IERC721EnumerableUpgradeable(settings.drawingToken).isApprovedForAll(owner, user) 
+                      || IERC721EnumerableUpgradeable(settings.drawingToken).getApproved(request.currentChosenTokenId) == user);            
    }
## Low & QA

1. fulfillRandomWords must not revert
2. transferFrom might return false instead of reverting
3. `redraw()` can be called if the network is congested and `request.drawTimelock` is low enough
4. There is no way for the owner to change the chainlink subscriptionId

### 1. fulfillRandomWords must not revert

The function `fulfillRandomWords()` at [VRFNFTRandomDraw.sol#L230](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L230) can revert but [chainlink docs](https://docs.chain.link/vrf/v2/security/#fulfillrandomwords-must-not-revert) states:

```
If your fulfillRandomWords() implementation reverts, the VRF service will not attempt to call it a second time. Make sure your contract logic does not revert. Consider simply storing the randomness and taking more complex follow-on actions in separate contract calls made by you, your users, or an Automation Node.
```

This could become an issue if chainlink takes more time than `request.drawTimelock` to produce a response, `redraw()` gets called again and then chainlink calls `fulfillRandomWords()` with the wrong `requestId`. No assets would be lost or locked thanks to `lastResortTimelockOwnerClaimNFT()`.

### 2. transferFrom might return false instead of reverting

At [VRFNFTRandomDraw.sol#L186](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L186) in `startDraw()` the function `transferFrom` is used to transfer an ERC721 from the owner to the contract itself, the function is wrapped in a `try/catch` statement which catches reverts but doesn't catch if `transferFrom` returns `false`, which might be the case for some ERC721 implementations.

The consequence is that the function `startDraw()` can execute succesfully even if the ERC721 token does not get transferred to the contract. There is no loss of assets but it would be appropriate to check both if `transferFrom` reverts and/or returns `false`.

### 3. `redraw()` can be called if the network is congested and `request.drawTimelock` is low enough

The minimum value accepted for `request.drawTimelock` is 1 hour. In case the network is congested the chainlink transaction could stay in the mempool for longer than 1 hour, an user could check if the winning number is it's own and call `redraw()` again to try his luck again. This issue is in-between a bug and a design choice, I'm reporting it to make sure the sponsor is aware.

### 4. There is no way for the owner to change the chainlink subscriptionId

In case the owner has set the wrong subscriptionId there is no way for him to change it, meaning in this case he would need to redeploy the contract losing some funds in extra gas costs.
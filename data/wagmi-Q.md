# Owner can immediately redraw after initalize()

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L206

## Impact
```solidity
function redraw() external onlyOwner returns (uint256) {
    if (request.drawTimelock >= block.timestamp) {
        revert TOO_SOON_TO_REDRAW();
    } // @audit can redraw immediately

    ...
}
```

After `initialize()`, owner can transfer the token directly to contract by using `transfer()` function. Then owner can call `redraw()` immediately without calling `startDraw()` first.

It will make users confuse because event SetupDraw is not emitted and it affect UI/UX

## Tools Used
Manual Review

## Recommended Mitigation Steps
Adding check to make sure `startDraw()` has to be called first.

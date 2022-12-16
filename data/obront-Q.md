# [L-1] InitializedDraw event emits wrong parameters

In VRFNFTRandomDraw.sol#initialize(), we emit the following event:

```solidity
emit InitializedDraw(msg.sender, settings);
```

However, this function is called by the Factory, so `msg.sender` will always be the Factory address.

## Recommended Mitigation
The original `msg.sender` is passed to this function as `admin`, so the event should be replaced with:

```solidity
emit InitializedDraw(admin, settings);
```

# [L-2] redraw() can be called for first draw

There is no check in `redraw()` that it is actually a redraw. The only different check is that `request.drawTimelock >= block.timestamp`, but when we are starting the first draw, `request.drawTimelock == 0`, which will pass this check.

Later in the function, we check that the contract already holds the NFT, but it would be easy for any user to transfer it in manually before calling `redraw()`.

This can't be used to cause any harm to the protocol, but we should be enforcing that users use the `startDraw()` function for their first draw.

## Recommended Mitigation
Add the following check at the top of `redraw()`:

```solidity
if (request.drawTimelock == 0) {
    revert CANNOT_REDRAW_BEFORE_DRAW();
}
```

# [QA-1] Unnecessary check on drawTimelock in _requestRoll()

In `_requestRoll()` we perform the following check:

```solidity
if (
    request.hasChosenRandomNumber &&
    request.drawTimelock != 0 &&
    request.drawTimelock > block.timestamp
) 
```

The `request.drawTimelock != 0` check is intended to check whether it's the first draw. However, in the situation where `request.drawTimelock == 0`, it will also be less than `block.timestamp`, so the additional check is unnecessary.

## Recommended Mitigation
Replace with:

```solidity
if (
    request.hasChosenRandomNumber &&
    request.drawTimelock > block.timestamp
) 
```

# [QA-2] Unnecessary check of request ID in startDraw()

When `startDraw()` is called, the first thing we do is the following check:

```solidity
if (request.currentChainlinkRequestId != 0) {
    revert REQUEST_IN_FLIGHT();
}
```

However, the next action we take is to call `_requestRoll()`, where this exact check is immediately done. There is no need to do it twice.

## Recommended Mitigation
Remove the check from the top of `startDraw()`.
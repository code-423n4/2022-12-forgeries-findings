# 1. Remove unnecessary comparison to zero && comparison to a positive number

Code diff

```diff
diff --git a/src/VRFNFTRandomDraw.sol b/src/VRFNFTRandomDraw.sol
index 668bc56..d5b53d3 100644
--- a/src/VRFNFTRandomDraw.sol
+++ b/src/VRFNFTRandomDraw.sol
@@ -149,7 +149,6 @@ contract VRFNFTRandomDraw is
         if (
             request.hasChosenRandomNumber &&
             // Draw timelock not yet used
-            request.drawTimelock != 0 &&
             request.drawTimelock > block.timestamp
         ) {
             revert STILL_IN_WAITING_PERIOD_BEFORE_REDRAWING();

```

Gas diff

```
test_LoserCannotWithdraw() (gas: -20 (-0.001%))
test_ValidateRequestID() (gas: -20 (-0.002%))
test_FullDrawing() (gas: -20 (-0.002%))
test_NFTNotApproved() (gas: -20 (-0.003%))
test_TokenNotApproved() (gas: -20 (-0.004%))
test_CannotRerollInFlight() (gas: -40 (-0.004%))
test_DrawingUserCheck() (gas: -40 (-0.004%))
Overall gas change: -180 (-0.021%)
```
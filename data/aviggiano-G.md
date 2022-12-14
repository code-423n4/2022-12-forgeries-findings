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

# 2. Change `immutable` state variables to `constant` integers


Code diff

```diff
diff --git a/src/VRFNFTRandomDraw.sol b/src/VRFNFTRandomDraw.sol
index 668bc56..f717cfb 100644
--- a/src/VRFNFTRandomDraw.sol
+++ b/src/VRFNFTRandomDraw.sol
@@ -19,18 +19,18 @@ contract VRFNFTRandomDraw is
     Version(1)
 {
     /// @notice Our callback is just setting a few variables, 200k should be more than enough gas.
-    uint32 immutable callbackGasLimit = 200_000;
+    uint32 constant callbackGasLimit = 200_000;
     /// @notice Chainlink request confirmations, left at the default
-    uint16 immutable minimumRequestConfirmations = 3;
+    uint16 constant minimumRequestConfirmations = 3;
     /// @notice Number of words requested in a drawing
-    uint16 immutable wordsRequested = 1;
+    uint16 constant wordsRequested = 1;
 
     /// @dev 60 seconds in a min, 60 mins in an hour
-    uint256 immutable HOUR_IN_SECONDS = 60 * 60;
+    uint256 constant HOUR_IN_SECONDS = 60 * 60;
     /// @dev 24 hours in a day 7 days in a week
-    uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
+    uint256 constant WEEK_IN_SECONDS = (3600 * 24 * 7);
     // @dev about 30 days in a month
-    uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
+    uint256 constant MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
 
 
     /// @notice Reference to chain-specific coordinator contract
```

Gas diff

```
test_LoserCannotWithdraw() (gas: -24 (-0.001%))
test_NFTNotApproved() (gas: -18 (-0.002%))
test_ValidateRequestID() (gas: -24 (-0.003%))
test_FullDrawing() (gas: -24 (-0.003%))
test_TokenNotApproved() (gas: -18 (-0.004%))
test_CannotRerollInFlight() (gas: -36 (-0.004%))
test_DrawingUserCheck() (gas: -48 (-0.005%))
Overall gas change: -192 (-0.022%)
```

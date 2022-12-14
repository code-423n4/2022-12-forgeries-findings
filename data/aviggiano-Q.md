# 1. Use Solidity time units for better readability of the code

See https://docs.soliditylang.org/en/v0.8.17/units-and-global-variables.html#time-units

```diff
diff --git a/src/VRFNFTRandomDraw.sol b/src/VRFNFTRandomDraw.sol
index 668bc56..3166545 100644
--- a/src/VRFNFTRandomDraw.sol
+++ b/src/VRFNFTRandomDraw.sol
@@ -25,12 +25,9 @@ contract VRFNFTRandomDraw is
     /// @notice Number of words requested in a drawing
     uint16 immutable wordsRequested = 1;
 
-    /// @dev 60 seconds in a min, 60 mins in an hour
-    uint256 immutable HOUR_IN_SECONDS = 60 * 60;
-    /// @dev 24 hours in a day 7 days in a week
-    uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
-    // @dev about 30 days in a month
-    uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
+    uint256 immutable HOUR_IN_SECONDS = 1 hours;
+    uint256 immutable WEEK_IN_SECONDS = 1 weeks;
+    uint256 immutable MONTH_IN_SECONDS = 30 days;
 
 
     /// @notice Reference to chain-specific coordinator contract

```
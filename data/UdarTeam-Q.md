# [Q-01] EXPLICITLY LABEL THE VISIBILITY OF STATE VARIABLES

Labeling the visibility explicitly will improve code quality and will make it easier to catch incorrect assumptions about who can access the variable.

contracts/VRFNFTRandomDraw: [L22](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L22), [L24](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L24), [L26](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L26), [L29](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L29), [L31](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L31), [L33](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L33), [L37](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L37)

# [Q-02] MISSING FORWARD SLASH SYMBOL IN NATSPEC FORMAT DOCUMENTATION

contracts/VRFNFTRandomDraw: [L32](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L32)

# [Q-03] USE CONSTANT INSTEAD OF IMMUTABLE

In ```VRFNFTRandomDraw.sol``` contract immutable state variables can be declares as constants, because they are known on compile time and not set in constructor.

```diff
diff --git a/src/VRFNFTRandomDraw.sol b/src/VRFNFTRandomDraw.sol
index 668bc56..d53020b 100644
--- a/src/VRFNFTRandomDraw.sol
+++ b/src/VRFNFTRandomDraw.sol
@@ -19,19 +19,18 @@ contract VRFNFTRandomDraw is
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
```

# [Q-04] MOVE MODIFIER TO [MODIFIERS](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L31-L33) SECTION

main/src/ownable/OwnableUpgradeable.sol: [24](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L24)

# [Q-05] TYPO IN FUNCTION COMMENT

main/src/VRFNFTRandomDraw.sol: [294](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L294) 

```diff
-        // Transfer token to the winter.
+        // Transfer token to the winner.
```

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

# 2. Include `owner` on initialization events so that they can be matched with finalizing events

The current code emits an `event InitializedDraw(address indexed sender, Settings settings)`, but does not include the `owner` in its signature, while `event OwnerReclaimedNFT(address indexed owner)`, for example, contains it. It is possible that the `sender` is not the same as the `owner`, so dApps watching for these logs would be unable to match the initialization/finalization of a raffle. In addition, the event parameter names can be changed from `sender` to `owner` to indicate in each case that the address belongs to the raffle owner.

```diff
diff --git a/src/VRFNFTRandomDraw.sol b/src/VRFNFTRandomDraw.sol
index 668bc56..26e1abd 100644
--- a/src/VRFNFTRandomDraw.sol
+++ b/src/VRFNFTRandomDraw.sol
@@ -120,7 +120,7 @@ contract VRFNFTRandomDraw is
         __Ownable_init(admin);
 
         // Emit initialized event for indexing
-        emit InitializedDraw(msg.sender, settings);
+        emit InitializedDraw(msg.sender, admin, settings);
 
         // Get owner of raffled tokenId and ensure the current owner is the admin
         try
diff --git a/src/interfaces/IVRFNFTRandomDraw.sol b/src/interfaces/IVRFNFTRandomDraw.sol
index 4775288..8f70ed7 100644
--- a/src/interfaces/IVRFNFTRandomDraw.sol
+++ b/src/interfaces/IVRFNFTRandomDraw.sol
@@ -40,9 +40,9 @@ interface IVRFNFTRandomDraw {
     error WRONG_LENGTH_FOR_RANDOM_WORDS();
 
     /// @notice When the draw is initialized
-    event InitializedDraw(address indexed sender, Settings settings);
+    event InitializedDraw(address indexed sender, address indexed owner, Settings settings);
     /// @notice When the draw is setup
-    event SetupDraw(address indexed sender, Settings settings);
+    event SetupDraw(address indexed owner, Settings settings);
     /// @notice When the owner reclaims nft aftr recovery time delay
     event OwnerReclaimedNFT(address indexed owner);
     /// @notice Dice roll is complete from callback

```
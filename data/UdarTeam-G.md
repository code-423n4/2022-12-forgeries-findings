# [G-01] EMIT FROM MEMORY VARIABLE

main/src/VRFNFTRandomDraw.sol: [123](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L123)

```diff
         // Emit initialized event for indexing
-        emit InitializedDraw(msg.sender, settings);
+        emit InitializedDraw(msg.sender, _settings);
```

# [G-02] MOVE CHECKS TO THE TOP OF FUNCTION TO SAVE GAS FROM UNNECESSARY COMPUTATION IF THEY FAILS

main/src/VRFNFTRandomDraw.sol: [126](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L126), [186](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L186), [215](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L215), 

```diff
-        // Set new settings
-        settings = _settings;
-
         // Check values in memory:
         if (_settings.drawBufferTime < HOUR_IN_SECONDS) {
             revert REDRAW_TIMELOCK_NEEDS_TO_BE_MORE_THAN_AN_HOUR();
@@ -115,13 +111,6 @@ contract VRFNFTRandomDraw is
         ) {
             revert DRAWING_TOKEN_RANGE_INVALID();
         }
-
-        // Setup owner as admin
-        __Ownable_init(admin);
-
-        // Emit initialized event for indexing
-        emit InitializedDraw(msg.sender, settings);
-
         // Get owner of raffled tokenId and ensure the current owner is the admin
         try
             IERC721EnumerableUpgradeable(_settings.token).ownerOf(
@@ -135,6 +124,18 @@ contract VRFNFTRandomDraw is
         } catch {
             revert TOKEN_BEING_OFFERED_NEEDS_TO_EXIST();
         }
+
+        // @audit-info // Moved this to the bottom of checks to save gas from SSTORE
+        // Set new settings
+        settings = _settings;
+
+        // Setup owner as admin
+        __Ownable_init(admin);
+
+        // @audit-info // emit the given _settings, not from storage
+        // Emit initialized event for indexing
+        emit InitializedDraw(msg.sender, _settings);

/////////////////////////////////////////////////////////////////////////////////

-        // Emit setup draw user event
-        emit SetupDraw(msg.sender, settings);
-
-        // Request initial roll
-        _requestRoll();
-
         // Attempt to transfer token into this address
         try
             IERC721EnumerableUpgradeable(settings.token).transferFrom(
@@ -193,6 +187,13 @@ contract VRFNFTRandomDraw is
             revert TOKEN_NEEDS_TO_BE_APPROVED_TO_CONTRACT();
         }
 
+        // Emit setup draw user event
+        emit SetupDraw(msg.sender, settings);
+
+        // Request initial roll
+        _requestRoll();
+
+
         // Return the current chainlink request id
         return request.currentChainlinkRequestId;
     }
@@ -204,13 +205,7 @@ contract VRFNFTRandomDraw is
         if (request.drawTimelock >= block.timestamp) {
             revert TOO_SOON_TO_REDRAW();
         }
-
-        // Reset request
-        delete request;
-
-        // Re-roll
-        _requestRoll();
-
+        
         // Owner of token to raffle needs to be this contract
         if (
             IERC721EnumerableUpgradeable(settings.token).ownerOf(
@@ -220,6 +215,12 @@ contract VRFNFTRandomDraw is
             revert DOES_NOT_OWN_NFT();
         }
 
+        // Reset request
+        delete request;
+
+        // Re-roll
+        _requestRoll();
+
         // Return current chainlink request ID
         return request.currentChainlinkRequestId;
```

# [G-03] USE INTERNAL FUNCTIONS INSTEAD OF MODIFIERS

main/src/ownable/OwnableUpgradeable.sol: [24](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L24), [36](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L36), [44](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L44)

```diff
-    modifier notZeroAddress(address check) {
+    function notZeroAddress(address check) internal {
         if (check == address(0)) {
             revert OWNER_CANNOT_BE_ZERO_ADDRESS();
         }
-        _;
     }
 
     ///                                                          ///
@@ -33,19 +32,17 @@ abstract contract OwnableUpgradeable is IOwnableUpgradeable, Initializable {
     ///                                                          ///
 
     /// @dev Ensures the caller is the owner
-    modifier onlyOwner() {
+    function onlyOwner() internal {
         if (msg.sender != _owner) {
             revert ONLY_OWNER();
         }
-        _;
     }
 
     /// @dev Ensures the caller is the pending owner
-    modifier onlyPendingOwner() {
+    function onlyPendingOwner() internal {
         if (msg.sender != _pendingOwner) {
             revert ONLY_PENDING_OWNER();
         }
-        _;
     }
```

## OVERALL GAS REPORT DIFF

```
test_LoserCannotWithdraw() (gas: -758 (-0.025%)) 
testFactoryDoesNotAllowZeroAddressInitalization() (gas: -40 (-0.067%)) 
test_DrawingUserCheck() (gas: -698 (-0.078%)) 
test_ValidateRequestID() (gas: -773 (-0.088%)) 
test_FullDrawing() (gas: -773 (-0.088%)) 
test_ResignOwnership() (gas: 24 (0.118%)) 
test_SafeTransferOwnership() (gas: 51 (0.145%)) 
test_OwnershipSetup() (gas: 24 (0.176%)) 
test_CancelsOwnershipTransfer() (gas: 57 (0.184%)) 
test_TransferOwnershipSimple() (gas: 48 (0.204%)) 
test_PendingThenTransfer() (gas: 76 (0.227%)) 
test_GatedOnlyAdmin() (gas: 39 (0.239%)) 
test_Version() (gas: -797 (-0.256%)) 
test_NotTransferOwnershipZero() (gas: 190 (1.115%)) 
testFactoryAttemptsToSetupChildContract() (gas: -40856 (-3.023%)) 
testFactoryInitializeProxy() (gas: -40856 (-3.139%)) 
testFactoryUpgrade() (gas: -122460 (-3.518%)) 
testFactoryInitializeConstructor() (gas: -40880 (-3.783%)) 
testFactoryVersion() (gas: -40880 (-3.798%)) 
test_CannotRerollInFlight() (gas: -88223 (-9.577%)) 
test_NFTNotApproved() (gas: -88271 (-11.114%)) 
test_TokenNotApproved() (gas: -88271 (-18.774%)) 
test_InvalidOptionTime() (gas: -139774 (-38.555%)) 
test_ZeroTokenContract() (gas: -59858 (-42.329%)) 
test_BadDrawingRange() (gas: -119558 (-43.082%)) 
test_InvalidRecoverTimelock() (gas: -59858 (-43.378%)) 
test_NoTokenOwner() (gas: -126217 (-58.123%)) 
```
```
Overall gas change: -1059292 (-280.387%)
```



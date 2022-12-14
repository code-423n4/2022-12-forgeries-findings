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

## OVERALL GAS REPORT DIFF

```Test result: ok. 13 passed; 0 failed; finished in 6.54ms
test_CancelsOwnershipTransfer() (gas: 0 (0.000%)) 
test_GatedOnlyAdmin() (gas: 0 (0.000%)) 
test_NotTransferOwnershipZero() (gas: 0 (0.000%)) 
test_OwnershipSetup() (gas: 0 (0.000%)) 
test_PendingThenTransfer() (gas: 0 (0.000%)) 
test_ResignOwnership() (gas: 0 (0.000%)) 
test_SafeTransferOwnership() (gas: 0 (0.000%)) 
test_TransferOwnershipSimple() (gas: 0 (0.000%)) 
testFactoryAttemptsToSetupChildContract() (gas: 0 (0.000%)) 
testFactoryDoesNotAllowZeroAddressInitalization() (gas: 0 (0.000%)) 
testFactoryInitializeConstructor() (gas: 0 (0.000%)) 
testFactoryInitializeProxy() (gas: 0 (0.000%)) 
testFactoryUpgrade() (gas: 0 (0.000%)) 
testFactoryVersion() (gas: 0 (0.000%)) 
test_LoserCannotWithdraw() (gas: -821 (-0.027%)) 
test_DrawingUserCheck() (gas: -818 (-0.091%)) 
test_ValidateRequestID() (gas: -821 (-0.093%)) 
test_FullDrawing() (gas: -821 (-0.094%)) 
test_Version() (gas: -821 (-0.264%)) 
test_CannotRerollInFlight() (gas: -88319 (-9.588%)) 
test_NFTNotApproved() (gas: -88319 (-11.120%)) 
test_TokenNotApproved() (gas: -88319 (-18.784%)) 
test_InvalidOptionTime() (gas: -139774 (-38.555%)) 
test_ZeroTokenContract() (gas: -59858 (-42.329%)) 
test_BadDrawingRange() (gas: -119558 (-43.082%)) 
test_InvalidRecoverTimelock() (gas: -59858 (-43.378%)) 
test_NoTokenOwner() (gas: -126217 (-58.123%)) 
```
```
Overall gas change: -774324 (-265.528%)
```



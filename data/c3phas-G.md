## FINDINGS
NB: Some functions have been truncated where necessary to just show affected parts of the code
Throughout the report some places might be denoted with audit tags to show the actual place affected.

- [FINDINGS](#findings)
- [Notes on Gas estimates.](#notes-on-gas-estimates)
- [Pack structs by putting data types in ascending size(We can save up to ~6k gas)](#pack-structs-by-putting-data-types-in-ascending-sizewe-can-save-up-to-6k-gas)
  - [We can use a smaller type for  uint256 drawTimelock as it's simply a timestamp. Using uint64 should be safe for 532 years. We save 1 Storage SLOT from 4 SLOTS to 3 SLOTS (~2K gas)](#we-can-use-a-smaller-type-for--uint256-drawtimelock-as-its-simply-a-timestamp-using-uint64-should-be-safe-for-532-years-we-save-1-storage-slot-from-4-slots-to-3-slots-2k-gas)
  - [We can save 2 SLOTs here by packing address token with uint64 subscriptionId and also changing the type of  uint256 recoverTimelock which is a timestamp to uint64 which should be safe for more than 500 years (Save ~4k gas)](#we-can-save-2-slots-here-by-packing-address-token-with-uint64-subscriptionid-and-also-changing-the-type-of--uint256-recovertimelock-which-is-a-timestamp-to-uint64-which-should-be-safe-for-more-than-500-years-save-4k-gas)
- [Emitting storage values instead of the memory one.](#emitting-storage-values-instead-of-the-memory-one)
  - [Save 499 gas on average](#save-499-gas-on-average)
- [Cache storage values in memory to minimize SLOADs](#cache-storage-values-in-memory-to-minimize-sloads)
  - [VRFNFTRandomDraw.sol.\_requestRoll(): We could cache request.drawTimelock instead of calling it twice](#vrfnftrandomdrawsol_requestroll-we-could-cache-requestdrawtimelock-instead-of-calling-it-twice)
  - [VRFNFTRandomDraw.sol.winnerClaimNFT(): settings.token and settings.tokenId should be cached. Also no need to cast settings.token as it's an address already - Save 62 gas on average](#vrfnftrandomdrawsolwinnerclaimnft-settingstoken-and-settingstokenid-should-be-cached-also-no-need-to-cast-settingstoken-as-its-an-address-already---save-62-gas-on-average)
- [The result of a function call should be cached rather than re-calling the function](#the-result-of-a-function-call-should-be-cached-rather-than-re-calling-the-function)
  - [VRFNFTRandomDraw.sol.lastResortTimelockOwnerClaimNFT(): The results of `owner()` should be cached instead of calling it twice](#vrfnftrandomdrawsollastresorttimelockownerclaimnft-the-results-of-owner-should-be-cached-instead-of-calling-it-twice)
- [Using unchecked blocks to save gas](#using-unchecked-blocks-to-save-gas)
  - [Save 43 gas on average](#save-43-gas-on-average)
- [A modifier used only once and not being inherited should be inlined to save gas](#a-modifier-used-only-once-and-not-being-inherited-should-be-inlined-to-save-gas)
- [Caching global variables is more expensive than using the actual variable(use msg.sender instead of caching it)](#caching-global-variables-is-more-expensive-than-using-the-actual-variableuse-msgsender-instead-of-caching-it)
- [Using `private` rather than `public` for constants, saves gas](#using-private-rather-than-public-for-constants-saves-gas)
- [Functions guaranteed to revert when called by normal users can be marked `payable`](#functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable)


## Notes on Gas estimates.
I've tried to give the exact amount of gas being saved from running the included tests. Whenever the function is within the test coverage, the average gas before and after will be included, and often a diff of the code will also accompany this.

Some functions are not covered by the test cases or are internal/private functions. In this case, the gas can be estimated by looking at the opcodes involved. 


## Pack structs by putting data types in ascending size(We can save up to ~6k gas)

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, this saves gas when writing to storage ~20000 gas

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L59-L68
### We can use a smaller type for  uint256 drawTimelock as it's simply a timestamp. Using uint64 should be safe for 532 years. We save 1 Storage SLOT from 4 SLOTS to 3 SLOTS (~2K gas)
```solidity
File: /src/interfaces/IVRFNFTRandomDraw.sol
59:    struct CurrentRequest {
60:        /// @notice current chainlink request id
61:        uint256 currentChainlinkRequestId;
62:        /// @notice current chosen random number
63:        uint256 currentChosenTokenId;
64:        /// @notice has chosen a random number (in case random number = 0(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0))
65:        bool hasChosenRandomNumber;
66:        /// @notice time lock (block.timestamp) that a re-draw can be issued
67:        uint256 drawTimelock;
68:    }
```


```diff
diff --git a/src/interfaces/IVRFNFTRandomDraw.sol b/src/interfaces/IVRFNFTRandomDraw.sol
index 4775288..af1d928 100644
--- a/src/interfaces/IVRFNFTRandomDraw.sol
+++ b/src/interfaces/IVRFNFTRandomDraw.sol
@@ -64,7 +64,7 @@ interface IVRFNFTRandomDraw {
         /// @notice has chosen a random number (in case random number = 0(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0))
         bool hasChosenRandomNumber;
         /// @notice time lock (block.timestamp) that a re-draw can be issued
-        uint256 drawTimelock;
+        uint64 drawTimelock;
     }
```


https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L71-L90
### We can save 2 SLOTs here by packing address token with uint64 subscriptionId and also changing the type of  uint256 recoverTimelock which is a timestamp to uint64 which should be safe for more than 500 years (Save ~4k gas)

```solidity
File: /src/interfaces/IVRFNFTRandomDraw.sol
71:    struct Settings {
72:        /// @notice Token Contract to put up for raffle
73:        address token;
74:        /// @notice Token ID to put up for raffle
75:        uint256 tokenId;
76:        /// @notice Token that each (sequential) ID has a entry in the raffle.
77:        address drawingToken;
78:        /// @notice Start token ID for the drawing (if totalSupply = 20 but the first token is 5 (5-25), setting this to 5 would fix the ordering)
79:        uint256 drawingTokenStartId;
80:        /// @notice End token ID for the drawing (exclusive) (token ids 0 - 9 would be 10 in this field)
81:        uint256 drawingTokenEndId;
82:        /// @notice Draw buffer time – time until a re-drawing can occur if the selected user cannot or does not claim the NFT.
83:        uint256 drawBufferTime;
84:        /// @notice block.timestamp that the admin can recover the NFT (as a safety fallback)
85:        uint256 recoverTimelock;
86:        /// @notice Chainlink gas keyhash
87:        bytes32 keyHash;
88:        /// @notice Chainlink subscription id
89:        uint64 subscriptionId;
90:    }
```

```diff
diff --git a/src/interfaces/IVRFNFTRandomDraw.sol b/src/interfaces/IVRFNFTRandomDraw.sol
index 4775288..7923c29 100644
--- a/src/interfaces/IVRFNFTRandomDraw.sol
+++ b/src/interfaces/IVRFNFTRandomDraw.sol
@@ -69,24 +69,24 @@ interface IVRFNFTRandomDraw {

     /// @notice Struct to organize user settings
     struct Settings {
+        /// @notice Chainlink subscription id
+        uint64 subscriptionId;
         /// @notice Token Contract to put up for raffle
         address token;
         /// @notice Token ID to put up for raffle
         uint256 tokenId;
         /// @notice Token that each (sequential) ID has a entry in the raffle.
         address drawingToken;
+        /// @notice block.timestamp that the admin can recover the NFT (as a safety fallback)
+        uint64 recoverTimelock;
         /// @notice Start token ID for the drawing (if totalSupply = 20 but the first token is 5 (5-25), setting this to 5 would fix the ordering)
         uint256 drawingTokenStartId;
         /// @notice End token ID for the drawing (exclusive) (token ids 0 - 9 would be 10 in this field)
         uint256 drawingTokenEndId;
         /// @notice Draw buffer time – time until a re-drawing can occur if the selected user cannot or does not claim the NFT.
         uint256 drawBufferTime;
-        /// @notice block.timestamp that the admin can recover the NFT (as a safety fallback)
-        uint256 recoverTimelock;
         /// @notice Chainlink gas keyhash
         bytes32 keyHash;
-        /// @notice Chainlink subscription id
-        uint64 subscriptionId;
     }
```
## Emitting storage values instead of the memory one.
Here, the values emitted shouldn’t be read from storage. The existing memory values should be used instead:

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L75-L138

### Save 499 gas on average

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 43790    | 146546   | 175523 | 192923 |
| After  | 43790    | 146047   | 174692 | 192092 |

```solidity
File: /src/VRFNFTRandomDraw.sol
75:    function initialize(address admin, Settings memory _settings)
76:        public
77:        initializer
78:    {
79:        // Set new settings
80:        settings = _settings;

122:        // Emit initialized event for indexing
123:        emit InitializedDraw(msg.sender, settings);
```

```diff
diff --git a/src/VRFNFTRandomDraw.sol b/src/VRFNFTRandomDraw.sol
index 668bc56..7955234 100644
--- a/src/VRFNFTRandomDraw.sol
+++ b/src/VRFNFTRandomDraw.sol
@@ -120,7 +120,7 @@ contract VRFNFTRandomDraw is
         __Ownable_init(admin);

         // Emit initialized event for indexing
-        emit InitializedDraw(msg.sender, settings);
+        emit InitializedDraw(msg.sender, _settings);

         // Get owner of raffled tokenId and ensure the current owner is the admin
```

## Cache storage values in memory to minimize SLOADs
The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L141-L169
### VRFNFTRandomDraw.sol.\_requestRoll(): We could cache request.drawTimelock instead of calling it twice
```solidity
File: /src/VRFNFTRandomDraw.sol
141:    function _requestRoll() internal {

148:        // If the number has been drawn and
149:        if (
150:            request.hasChosenRandomNumber &&
151:            // Draw timelock not yet used
152:            request.drawTimelock != 0 && //@audit: 1st call 
153:            request.drawTimelock > block.timestamp //@audit: 2nd call
154:        ) {
155:            revert STILL_IN_WAITING_PERIOD_BEFORE_REDRAWING();
156:        }

158:        // Setup re-draw timelock
159:        request.drawTimelock = block.timestamp + settings.drawBufferTime;

169:    }
```

```diff
diff --git a/src/VRFNFTRandomDraw.sol b/src/VRFNFTRandomDraw.sol
index 668bc56..e235c0b 100644
--- a/src/VRFNFTRandomDraw.sol
+++ b/src/VRFNFTRandomDraw.sol
@@ -145,12 +145,13 @@ contract VRFNFTRandomDraw is
             revert REQUEST_IN_FLIGHT();
         }

-        // If the number has been drawn and
+        // If the number has been drawn
+        uint256 _requestDrawTimelock = request.drawTimelock ;
         if (
             request.hasChosenRandomNumber &&
             // Draw timelock not yet used
-            request.drawTimelock != 0 &&
-            request.drawTimelock > block.timestamp
+            _requestDrawTimelock != 0 &&
+            _requestDrawTimelock > block.timestamp
         ) {
             revert STILL_IN_WAITING_PERIOD_BEFORE_REDRAWING();
         }
```


https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L277-L300
### VRFNFTRandomDraw.sol.winnerClaimNFT(): settings.token and settings.tokenId should be cached. Also no need to cast settings.token as it's an address already - Save 62 gas on average


|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 422    | 11638   | 2422 | 27624 |
| After  | 422    | 11576   | 2422 | 27469 |

```solidity
File: /src/VRFNFTRandomDraw.sol
277:    function winnerClaimNFT() external {

287:        emit WinnerSentNFT(
288:            user,
289:            address(settings.token),
290:            settings.tokenId,
291:            settings
292:        );

294:        // Transfer token to the winter.
295:        IERC721EnumerableUpgradeable(settings.token).transferFrom(
296:            address(this),
297:            msg.sender,
298:            settings.tokenId
299:        );
300:    }
```

```diff
diff --git a/src/VRFNFTRandomDraw.sol b/src/VRFNFTRandomDraw.sol
index 668bc56..407a5f4 100644
--- a/src/VRFNFTRandomDraw.sol
+++ b/src/VRFNFTRandomDraw.sol
@@ -283,19 +283,21 @@ contract VRFNFTRandomDraw is
             revert USER_HAS_NOT_WON();
         }

+        address _token = settings.token;
+        uint256 _tokenId = settings.tokenId;
         // Emit a celebratory event
         emit WinnerSentNFT(
             user,
-            address(settings.token),
-            settings.tokenId,
+            _token,
+            _tokenId,
             settings
         );

         // Transfer token to the winter.
-        IERC721EnumerableUpgradeable(settings.token).transferFrom(
+        IERC721EnumerableUpgradeable(_token).transferFrom(
             address(this),
             msg.sender,
-            settings.tokenId
+            _tokenId
         );
     }
```

## The result of a function call should be cached rather than re-calling the function

Consider caching the following:

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L304-L320
### VRFNFTRandomDraw.sol.lastResortTimelockOwnerClaimNFT(): The results of `owner()` should be cached instead of calling it twice

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 381    | 11061   | 11061 | 21741 |
| After  | 381    | 10992   | 10992 | 21604 |

```solidity
File: /src/VRFNFTRandomDraw.sol
304:    function lastResortTimelockOwnerClaimNFT() external onlyOwner {
305:        // If recoverTimelock is not setup, or if not yet occurred
306:        if (settings.recoverTimelock > block.timestamp) {
307:            // Stop the withdraw
308:            revert RECOVERY_IS_NOT_YET_POSSIBLE();
309:        }

311:        // Send event for indexing that the owner reclaimed the NFT
312:        emit OwnerReclaimedNFT(owner()); //@audit: Initial call

314:        // Transfer token to the admin/owner.
315:        IERC721EnumerableUpgradeable(settings.token).transferFrom(
316:            address(this),
317:            owner(),//@audit: Second call
318:            settings.tokenId
319:        );
320:    }
```

```diff
diff --git a/src/VRFNFTRandomDraw.sol b/src/VRFNFTRandomDraw.sol
index 668bc56..00f000d 100644
--- a/src/VRFNFTRandomDraw.sol
+++ b/src/VRFNFTRandomDraw.sol
@@ -307,14 +307,15 @@ contract VRFNFTRandomDraw is
             // Stop the withdraw
             revert RECOVERY_IS_NOT_YET_POSSIBLE();
         }
+        address _ownerAddr = owner();

         // Send event for indexing that the owner reclaimed the NFT
-        emit OwnerReclaimedNFT(owner());
+        emit OwnerReclaimedNFT(_ownerAddr);

         // Transfer token to the admin/owner.
         IERC721EnumerableUpgradeable(settings.token).transferFrom(
             address(this),
-            owner(),
+            _ownerAddr,
             settings.tokenId
         );
     }
```

## Using unchecked blocks to save gas
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block
[see resource](https://github.com/ethereum/solidity/issues/10695)

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L112-L115
### Save 43 gas on average

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 43790    | 146546   | 175523 | 192923 |
| After  | 43790    | 146503   | 175451 | 192851 |

```solidity
File: /src/VRFNFTRandomDraw.sol
112:        if (
113:            _settings.drawingTokenEndId < _settings.drawingTokenStartId ||
114:            _settings.drawingTokenEndId - _settings.drawingTokenStartId < 2
115:        ) {
```

The operation ` _settings.drawingTokenEndId - _settings.drawingTokenStartId` cannot underflow as it would only be performed if the operation `_settings.drawingTokenEndId < _settings.drawingTokenStartId` is false(Short circuit rules)

```diff
diff --git a/src/VRFNFTRandomDraw.sol b/src/VRFNFTRandomDraw.sol
index 668bc56..b33b93e 100644
--- a/src/VRFNFTRandomDraw.sol
+++ b/src/VRFNFTRandomDraw.sol
@@ -109,12 +109,15 @@ contract VRFNFTRandomDraw is

         // Validate token range: end needs to be greater than start
         // and the size of the range needs to be at least 2 (end is exclusive)
-        if (
+        unchecked {
+            if (
             _settings.drawingTokenEndId < _settings.drawingTokenStartId ||
             _settings.drawingTokenEndId - _settings.drawingTokenStartId < 2
         ) {
             revert DRAWING_TOKEN_RANGE_INVALID();
         }
+        }
+
```

## A modifier used only once and not being inherited should be inlined to save gas

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/ownable/OwnableUpgradeable.sol#L44-L49
```solidity
File: /src/ownable/OwnableUpgradeable.sol
44:    modifier onlyPendingOwner() {
45:        if (msg.sender != _pendingOwner) {
46:            revert ONLY_PENDING_OWNER();
47:        }
48:        _;
49:    }
```

The above modifer is only used in the following
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/ownable/OwnableUpgradeable.sol#L119-L125
```solidity
File: /src/ownable/OwnableUpgradeable.sol
119:    function acceptOwnership() public onlyPendingOwner {
120:        emit OwnerUpdated(_owner, msg.sender);

122:        _owner = _pendingOwner;

124:        delete _pendingOwner;
125:    }
```

```diff
diff --git a/src/ownable/OwnableUpgradeable.sol b/src/ownable/OwnableUpgradeable.sol
index bfc7eef..d27530c 100644
--- a/src/ownable/OwnableUpgradeable.sol
+++ b/src/ownable/OwnableUpgradeable.sol
@@ -116,7 +116,10 @@ abstract contract OwnableUpgradeable is IOwnableUpgradeable, Initializable {
     }

     /// @notice Accepts an ownership transfer
-    function acceptOwnership() public onlyPendingOwner {
+    function acceptOwnership() public{
+      if (msg.sender != _pendingOwner) {
+            revert ONLY_PENDING_OWNER();
+        }
         emit OwnerUpdated(_owner, msg.sender);

         _owner = _pendingOwner;

		  delete _pendingOwner;     
```

## Caching global variables is more expensive than using the actual variable(use msg.sender instead of caching it)
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L38-L51

|        | Min    | Average | Median   | Max   |
| ------ | --- | ------- | ----- | ----- |
| Before | 46872    | 183639   | 213232 | 239795 |
| After  | 46860    | 183631   | 213224 | 239783 |

```solidity
File: /src/VRFNFTRandomDrawFactory.sol
38:    function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)
39:        external
40:        returns (address)
41:    {
42:        address admin = msg.sender;
43:        // Clone the contract
44:        address newDrawing = ClonesUpgradeable.clone(implementation);
45:        // Setup the new drawing
46:        IVRFNFTRandomDraw(newDrawing).initialize(admin, settings);
47:        // Emit event for indexing
48:        emit SetupNewDrawing(admin, newDrawing);
49:        // Return address for integration or testing
50:        return newDrawing;
51:    }
```

```diff
diff --git a/src/VRFNFTRandomDrawFactory.sol b/src/VRFNFTRandomDrawFactory.sol
index 84caedb..616cb0a 100644
--- a/src/VRFNFTRandomDrawFactory.sol
+++ b/src/VRFNFTRandomDrawFactory.sol
@@ -39,13 +39,12 @@ contract VRFNFTRandomDrawFactory is
         external
         returns (address)
     {
-        address admin = msg.sender;
         // Clone the contract
         address newDrawing = ClonesUpgradeable.clone(implementation);
         // Setup the new drawing
-        IVRFNFTRandomDraw(newDrawing).initialize(admin, settings);
+        IVRFNFTRandomDraw(newDrawing).initialize(msg.sender, settings);
         // Emit event for indexing
-        emit SetupNewDrawing(admin, newDrawing);
+        emit SetupNewDrawing(msg.sender, newDrawing);
         // Return address for integration or testing
         return newDrawing;
     }
```

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L277-L300

```solidity
File: /src/VRFNFTRandomDraw.sol
277:    function winnerClaimNFT() external {
278:        // Assume (potential) winner calls this fn, cache.
279:        address user = msg.sender;

281:        // Check if this user has indeed won.
282:        if (!hasUserWon(user)) {
283:            revert USER_HAS_NOT_WON();
284:        }

286:        // Emit a celebratory event
287:        emit WinnerSentNFT(
288:            user,
289:            address(settings.token),
290:            settings.tokenId,
291:            settings
292:        );

294:        // Transfer token to the winter.
295:        IERC721EnumerableUpgradeable(settings.token).transferFrom(
296:            address(this),
297:            msg.sender,
298:            settings.tokenId
299:        );
300:    }
```

```diff
diff --git a/src/VRFNFTRandomDraw.sol b/src/VRFNFTRandomDraw.sol
index 668bc56..06ae5b2 100644
--- a/src/VRFNFTRandomDraw.sol
+++ b/src/VRFNFTRandomDraw.sol
@@ -276,16 +276,15 @@ contract VRFNFTRandomDraw is
     /// @notice Function for the winner to call to retrieve their NFT
     function winnerClaimNFT() external {
         // Assume (potential) winner calls this fn, cache.
-        address user = msg.sender;

         // Check if this user has indeed won.
-        if (!hasUserWon(user)) {
+        if (!hasUserWon(msg.sender)) {
             revert USER_HAS_NOT_WON();
         }

         // Emit a celebratory event
         emit WinnerSentNFT(
-            user,
+            msg.sender,
             address(settings.token),
             settings.tokenId,
             settings
```

## Using `private` rather than `public` for constants, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. 

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L21
```solidity
File: /src/VRFNFTRandomDrawFactory.sol
21:    address public immutable implementation;
```
## Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.The extra opcodes avoided costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost

https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L173
```solidity
File: /src/VRFNFTRandomDraw.sol
173:    function startDraw() external onlyOwner returns (uint256) {

203:    function redraw() external onlyOwner returns (uint256) {

304:    function lastResortTimelockOwnerClaimNFT() external onlyOwner {
```
## Unused imports
Importing source units that are not referenced in the contract increases the size of deployment. Consider removing them to save some gas. If there was a plan to use them in the future, a comment explaining why they were imported would be helpful.

Here are the instances entailed:

[File: VRFNFTRandomDrawFactory.sol#L4](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L4)

```diff
- import {IERC721EnumerableUpgradeable} from "@openzeppelin/contracts-upgradeable/token/ERC721/extensions/IERC721EnumerableUpgradeable.sol";
```
[File: VRFNFTRandomDraw.sol#L8](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L8)

```diff
- import {VRFCoordinatorV2, VRFCoordinatorV2Interface} from "@chainlink/contracts/src/v0.8/VRFCoordinatorV2.sol";
+ import {VRFCoordinatorV2Interface} from "@chainlink/contracts/src/v0.8/VRFCoordinatorV2.sol";
```
## Use of named returns for local variables saves gas
You can have further advantages in term of gas cost by simply using named return values as temporary local variable.

For instance, the code block below may be refactored as follows:

[File: VRFNFTRandomDraw.sol#L173-L198](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L173-L198)

```diff
173: -    function startDraw() external onlyOwner returns (uint256) {
     +    function startDraw() external onlyOwner returns (uint256 _requestId) {
...
197: -        return request.currentChainlinkRequestId;
     +        _requestId = request.currentChainlinkRequestId;
```
## Unneeded Cache
Caching a global variable, `msg.sender` is unnecessary as it has no gas saving benefit in doing so.

Here is an instance entailed:

[File: VRFNFTRandomDrawFactory.sol#L42](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L42)

```
        address admin = msg.sender;
```
## `||` costs less gas than its equivalent `&&`
Rule of thumb: `(x && y)` is `(!(!x || !y))`

Even with the 10k Optimizer enabled: `||`, OR conditions cost less than their equivalent `&&`, AND conditions.

As an example, the instance below may be refactored as follows:

[File: VRFNFTRandomDraw.sol#L149-L154](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L149-L154)

```diff
-        if (
+        if (!(
-            request.hasChosenRandomNumber &&
+            !request.hasChosenRandomNumber ||
            // Draw timelock not yet used
-            request.drawTimelock != 0 &&
+            request.drawTimelock == 0 ||
-            request.drawTimelock > block.timestamp
+            request.drawTimelock <= block.timestamp
-        ) {
+        )) {
```
## Payable access control functions costs less gas
Consider marking functions with access control as `payable`. This will save 20 gas on each call by their respective permissible callers for not needing to have the compiler check for `msg.value`.

For instance, the code line below may be refactored as follows:

[File: VRFNFTRandomDraw.sol#L173](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L173)

```diff
-    function startDraw() external onlyOwner returns (uint256) {
+    function startDraw() external payable onlyOwner returns (uint256) {
```
## Memory variable not utilized when emitting event
In `VRFNFTRandomDraw.sol`, the emitted event is found to be using the `state variable` equivalent instead of the stack `memory variable`. Consider having the affected code line refactored as follows to save more gas as on the function call:

[File: VRFNFTRandomDraw.sol#L75-L138](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L75-L138)

```diff
123: -        emit InitializedDraw(msg.sender, settings);
     +        emit InitializedDraw(msg.sender, _settings);
```
## State Variables Repeatedly Read Should be Cached
SLOADs cost 100 gas each after the 1st one whereas MLOADs/MSTOREs only incur 3 gas each. As such, storage values read multiple times should be cached in the stack memory the first time (costing only 1 SLOAD) and then re-read from this cache to avoid multiple SLOADs.

For instance, `settings` in the code block below should be cached as follows:

[File: VRFNFTRandomDraw.sol#L277-L300](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L277-L300)

```diff
+       Settings memory _settings = settings // SLOAD 1
        emit WinnerSentNFT(
            user,
-            address(settings.token),
+            address(_settings.token), // MLOAD 1
-            settings.tokenId,
+            _settings.tokenId, // MLOAD 2
-            settings
+            _settings // MLOAD 3
        );

        // Transfer token to the winter.
-        IERC721EnumerableUpgradeable(settings.token).transferFrom(
+        IERC721EnumerableUpgradeable(_settings.token).transferFrom( // MLOAD 4
            address(this),
            msg.sender,
-            settings.tokenId
+            _settings.tokenId // MLOAD 5
        );
```
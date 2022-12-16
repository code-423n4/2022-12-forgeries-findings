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
- 173:    function startDraw() external onlyOwner returns (uint256) {
     +    function startDraw() external onlyOwner returns (uint256 _requestId) {
...
- 197:        return request.currentChainlinkRequestId;
     +        _requestId = request.currentChainlinkRequestId;
```
## Unneeded Cache
Caching a global variable, `msg.sender` is unnecessary as it has no gas saving benefit in doing so.

Here are the instances entailed:

[File: VRFNFTRandomDrawFactory.sol#L42](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L42)

```
        address admin = msg.sender;
```
[File: VRFNFTRandomDraw.sol#L279](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L279)

```
        address user = msg.sender;
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
## Redundant if blocks
The following if block of `startDraw()` in `VRFNFTRandomDraw.sol` may be removed considering the identical check is going to be executed on [lines 144 - 146](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L144-L146) when internally calling `_requestRoll()`, to save gas both on contract deployment and function call:

[File: VRFNFTRandomDraw.sol#L175-L177](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L175-L177)

```diff
-        if (request.currentChainlinkRequestId != 0) {
-            revert REQUEST_IN_FLIGHT();
-        }
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
- 123:        emit InitializedDraw(msg.sender, settings);
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
## Private function with embedded modifier reduces contract size
Consider having the logic of a modifier embedded through a private function to reduce contract size if need be. A private visibility that saves more gas on function calls than the internal visibility is adopted because the modifier will only be making this call inside the contract.

For instance, the modifier instance below may be refactored as follows:

[File: OwnableUpgradeable.sol#L44-L49](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L44-L49)

```diff
+    function _onlyPendingOwner() private view {
+        if (msg.sender != _pendingOwner) {
+            revert ONLY_PENDING_OWNER();
+        }
+    }

    modifier onlyPendingOwner() {
-        if (msg.sender != _pendingOwner) {
-            revert ONLY_PENDING_OWNER();
-        }
+        _onlyPendingOwner();
        _;
    }
```
## Function order affects gas consumption
The order of function will also have an impact on gas consumption. Because in smart contracts, there is a difference in the order of the functions. Each position will have an extra 22 gas. The order is dependent on method ID. So, if you rename the frequently accessed function to more early method ID, you can save gas cost. Please visit the following site for further information:

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

## Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
  version: "0.8.16",
  settings: {
    optimizer: {
      enabled: true,
      runs: 1000,
    },
  },
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

## Unchecked SafeMath saves gas
"Checked" math, which is default in ^0.8.0 is not free. The compiler will add some overflow checks, somehow similar to those implemented by `SafeMath`. While it is reasonable to expect these checks to be less expensive than the current `SafeMath`, one should keep in mind that these checks will increase the cost of "basic math operation" that were not previously covered. This particularly concerns variable increments in for loops. When no arithmetic overflow/underflow is going to happen, `unchecked { ... }` to use the previous wrapping behavior further saves gas just as in the instance entailed below, as an example:

[File: VRFNFTRandomDraw.sol#L249-L256](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L249-L256)

```diff
+ unchecked {
        uint256 tokenRange = settings.drawingTokenEndId -
            settings.drawingTokenStartId;

        // Store a number from it here (reduce number here to reduce gas usage)
        // We know there will only be 1 word sent at this point.
        request.currentChosenTokenId =
            (_randomWords[0] % tokenRange) +
            settings.drawingTokenStartId;
+ }
```
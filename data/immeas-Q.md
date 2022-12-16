# low

## L-1 draw can be started with `redraw()` instead of `startDraw()`

By transfering the raffled NFT to the contract manually a owner can start the raffle with `redraw()` instead of `startDraw()`.

### Impact
This omits sending the `SetupDraw` event which could affect listeners to the protocol. It also breaks the intended usage flow.

### Proof of Concept
PoC test in `VRFNFTRandomDraw.t.sol`:

```javascript
   function test_FullDrawingStartWithRedraw() public {
        address winner = address(0x1337);
        vm.label(winner, "winner");

        vm.startPrank(winner);
        for (uint256 tokensCount = 0; tokensCount < 10; tokensCount++) {
            drawingNFT.mint();
        }
        vm.stopPrank();

        vm.startPrank(admin);
        targetNFT.mint();        

        address consumerAddress = factory.makeNewDraw(
            IVRFNFTRandomDraw.Settings({
                token: address(targetNFT),
                tokenId: 0,
                drawingToken: address(drawingNFT),
                drawingTokenStartId: 0,
                drawingTokenEndId: 10,
                drawBufferTime: 1 hours,
                recoverTimelock: block.timestamp + 1 weeks,
                keyHash: bytes32(
                    0x79d3d8832d904592c0bf9818b621522c988bb8b0c05cdc3b15aea1b6e8db0c15
                ),
                subscriptionId: subscriptionId
            })
        );
        vm.label(consumerAddress, "drawing instance");

        mockCoordinator.addConsumer(subscriptionId, consumerAddress);
        mockCoordinator.fundSubscription(subscriptionId, 100 ether);

        VRFNFTRandomDraw drawing = VRFNFTRandomDraw(consumerAddress);
        targetNFT.setApprovalForAll(consumerAddress, true);

        // move nft to contract manually
        targetNFT.transferFrom(admin, consumerAddress, 0); 
        // start with redraw instead of startDraw()
        uint256 drawingId = drawing.redraw();

        mockCoordinator.fulfillRandomWords(drawingId, consumerAddress);
        vm.stopPrank();
        
        vm.prank(winner);
        drawing.winnerClaimNFT();
        assertEq(targetNFT.balanceOf(winner), 1);
        assertEq(targetNFT.balanceOf(consumerAddress), 0); 
    }
```

### Tools Used
vs code, forge

### Recommended Mitigation Steps
before doing a `redraw()`, verify that `currentChainlinkRequestId` is not empty. Then has to have been started before (through `startDraw()`).

## L-2 weird comment

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L64

```javascript
File: interfaces/IVRFNFTRandomDraw.sol

64:        /// @notice has chosen a random number (in case random number = 0(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0))
```

## L-3 typos in comments

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L46

```javascript
File: interfaces/IVRFNFTRandomDraw.sol

46:    /// @notice When the owner reclaims nft aftr recovery time delay
```
_aftr_

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L294

```javascript
File: VRFNFTRandomDraw.sol

294:        // Transfer token to the winter.
```
_winter_


## L-4 anyone can call `initialize()`

There is no validation that it's the factory that created the contract that is calling `initialize()`. This could be achieved by adding immutable value with the creator in the constructor.  

# non-crit

## NC-1 unreachable code #1

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L144-L146

In `_requestRoll()`:
```javascript
141:    function _requestRoll() internal {
142:        // Chainlink request cannot be currently in flight.
143:        // Request is cleared in re-roll if conditions are correct.
144:        if (request.currentChainlinkRequestId != 0) { // can never be !=0
145:            revert REQUEST_IN_FLIGHT();
146:        }
```

This can never happen. `_requestRoll()` is called from two places:

`startDraw()` which checks that `currentChainlinkRequestId != 0` before going into `_requestRoll()`:
```javascript
173:    function startDraw() external onlyOwner returns (uint256) {
174:        // Only can be called on first drawing
175:        if (request.currentChainlinkRequestId != 0) {
176:            revert REQUEST_IN_FLIGHT();
177:        }

...

182:        // Request initial roll
183:        _requestRoll();
```

and in `redraw()` which just before does `delete request` which resets `currentChainlinkRequestId` to `0`:
```javascript
203:    function redraw() external onlyOwner returns (uint256) {
204:        if (request.drawTimelock >= block.timestamp) {
205:            revert TOO_SOON_TO_REDRAW();
206:        }
207:
208:        // Reset request
209:        delete request;
210:
211:        // Re-roll
212:        _requestRoll();
```

## NC-2 unreachable code #2

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L149-L156

In `_requestRoll()`:
```javascript
203:        if (
204:            request.hasChosenRandomNumber &&
205:            // Draw timelock not yet used
206:            request.drawTimelock != 0 &&
207:            request.drawTimelock > block.timestamp
208:        ) {
209:            revert STILL_IN_WAITING_PERIOD_BEFORE_REDRAWING();
210:        }
```

This can never happen. `_requestRoll()` is called from these two places:

`startDraw()`, already has a check for that `hasChosenRandomNumber` is `0`:

```javascript
173:    function startDraw() external onlyOwner returns (uint256) {
174:        // Only can be called on first drawing
175:        if (request.currentChainlinkRequestId != 0) {
176:            revert REQUEST_IN_FLIGHT();
177:        }

...

182:        // Request initial roll
183:        _requestRoll();
```

and in `redraw()` `request.drawTimelock` is checked already and it then does `delete request` which resets `currentChainlinkRequestId` to `0`:
```javascript
203:    function redraw() external onlyOwner returns (uint256) {
204:        if (request.drawTimelock >= block.timestamp) {
205:            revert TOO_SOON_TO_REDRAW();
206:        }
207:
208:        // Reset request
209:        delete request;
210:
211:        // Re-roll
212:        _requestRoll();
```
## Use constant instead of immutable 

Within the `VRFNFTRandomDraw.sol` contract, it is cheaper to declare a number of state variables as `constant` rather than `immutable`.

These variables are:

```
callbackGasLimit
minimumRequestConfirmations
wordsRequested
HOUR_IN_SECONDS
WEEK_IN_SECONDS
MONTH_IN_SECONDS
```

Doing so, will save a small amount of gas.

## No need to create variables for time 

Solidity provides time units variables. which can be used to represent hours, days and weeks, etc. Rather than creating state variables with these values, the global time units can be used in the code directly. This will lead to saving 1104 gas.

Below is shown an updated extract from the `initialize` function of `VRFNFTRandomDraw.sol`.

```
function initialize(address admin, Settings memory _settings)
        public
        initializer
    {
        // Set new settings
        settings = _settings;

        // Check values in memory:
        if (_settings.drawBufferTime < 1 hours) {
            revert REDRAW_TIMELOCK_NEEDS_TO_BE_MORE_THAN_AN_HOUR();
        }
        if (_settings.drawBufferTime > 1 days * 30) {
            revert REDRAW_TIMELOCK_NEEDS_TO_BE_LESS_THAN_A_MONTH();
        }

        if (_settings.recoverTimelock < block.timestamp + 1 weeks) {
            revert RECOVER_TIMELOCK_NEEDS_TO_BE_AT_LEAST_A_WEEK();
        }
        if (
            _settings.recoverTimelock >
            block.timestamp + (1 days * 30 * 12)
        ) {
            revert RECOVER_TIMELOCK_NEEDS_TO_BE_LESS_THAN_A_YEAR();
        }
[...]
```

## Redundant check on `_requestRoll`

Within the `_requestRoll` function of `VRFNFTRandomDraw.sol`, there is one redundant check which can be safely removed:

```
        // If the number has been drawn and
        if (
            request.hasChosenRandomNumber &&
            // Draw timelock not yet used
            request.drawTimelock != 0 && 
            request.drawTimelock > block.timestamp
        ) {
            revert STILL_IN_WAITING_PERIOD_BEFORE_REDRAWING();
        }

```

In more details,`request.drawTimelock != 0` can be safely removed to save a little big of gas. This check is redundant due to the following check: `request.drawTimelock > block.timestamp`. During the first call of `_requestRoll`, we can be sure that the `settings.drawTimelock` is set to zero (default value for uint256), and then updated to be at least greater than the `block.timestamp` by at least 1 hour (minimum value for `settings.drawBufferTime`). 

## Redundant check in `fulfillRandomness`

The check that the length of random numbers sent by the VRF seems to be redundant as the request is for 1 number only (found in `_requestRoll`). The function already checks that the request ID matches and accesses the random word using the index 0, thus making the check redundant.

```
       // Validate number of words returned
        // Words requested is an immutable set to 1
        if (_randomWords.length != wordsRequested) {
            revert WRONG_LENGTH_FOR_RANDOM_WORDS();
        }
```

## Accessing msg.sender directly saves gas in certain situations

There are certain instances where using `msg.sender` directly rather than storing it in a local variable first will save gas. These are detailed below:

`winnerClaimNFT` of `VRFNFTRandomDraw`

```
  function winnerClaimNFT() external {
        // Assume (potential) winner calls this fn, cache.
        address user = msg.sender;

        [...]

        // Emit a celebratory event
        emit WinnerSentNFT(
            user,
            address(settings.token),
            settings.tokenId,
            settings
        );

        [...]
    }
```

When calling the event presented above, using `msg.sender` will save 20 gas. 

`makeNewDraw` of `VRFNFTRandomDrawFactory`

```
    /// @notice Function to make a new drawing
    /// @param settings settings for the new drawing
    function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)
        external
        returns (address)
    {
        // address admin = msg.sender;
        // Clone the contract
        address newDrawing = ClonesUpgradeable.clone(implementation);
        // Setup the new drawing
        IVRFNFTRandomDraw(newDrawing).initialize(msg.sender, settings); // @audit GAS directly passing msg.sender saves gas
        // Emit event for indexing
        emit SetupNewDrawing(msg.sender, newDrawing); // @audit GAS directly passing msg.sender saves gas
        // Return address for integration or testing
        return newDrawing;
    }
```

As shown above, the `msg.sender` was not stored on a local variable and inserted directly into the initialize function and event emission. 
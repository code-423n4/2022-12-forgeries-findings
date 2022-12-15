## Summary
In total, `1` gas optimization is found.

## Optimiaztions
1. use `private constant` for constants instead of `immutable` to save gas

_Instance (6)_

```
File: src/VRFNFTRandomDraw.sol, line 21 - 33

/// @notice Our callback is just setting a few variables, 200k should be more than enough gas.
uint32 immutable callbackGasLimit = 200_000;
/// @notice Chainlink request confirmations, left at the default
uint16 immutable minimumRequestConfirmations = 3;
/// @notice Number of words requested in a drawing
uint16 immutable wordsRequested = 1;

/// @dev 60 seconds in a min, 60 mins in an hour
uint256 immutable HOUR_IN_SECONDS = 60 * 60;
/// @dev 24 hours in a day 7 days in a week
uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
// @dev about 30 days in a month
uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```

`forge test` gas usage comparison

Before: (with `immutable`)
```
Running 13 tests for test/VRFNFTRandomDraw.t.sol:VRFNFTRandomDrawTest
[PASS] test_BadDrawingRange() (gas: 277512)
[PASS] test_CannotRerollInFlight() (gas: 921174)
[PASS] test_DrawingUserCheck() (gas: 896254)
[PASS] test_FullDrawing() (gas: 876483)
[PASS] test_InvalidOptionTime() (gas: 362535)
[PASS] test_InvalidRecoverTimelock() (gas: 137991)
[PASS] test_LoserCannotWithdraw() (gas: 2994515)
[PASS] test_NFTNotApproved() (gas: 794261)
[PASS] test_NoTokenOwner() (gas: 217155)
[PASS] test_TokenNotApproved() (gas: 470188)
[PASS] test_ValidateRequestID() (gas: 880060)
[PASS] test_Version() (gas: 311094)
[PASS] test_ZeroTokenContract() (gas: 141411)
Test result: ok. 13 passed; 0 failed; finished in 3.94ms
```

After (with `private constant`)
```
Running 13 tests for test/VRFNFTRandomDraw.t.sol:VRFNFTRandomDrawTest
[PASS] test_BadDrawingRange() (gas: 277512)
[PASS] test_CannotRerollInFlight() (gas: 921138)
[PASS] test_DrawingUserCheck() (gas: 896206)
[PASS] test_FullDrawing() (gas: 876459)
[PASS] test_InvalidOptionTime() (gas: 362535)
[PASS] test_InvalidRecoverTimelock() (gas: 137991)
[PASS] test_LoserCannotWithdraw() (gas: 2994491)
[PASS] test_NFTNotApproved() (gas: 794243)
[PASS] test_NoTokenOwner() (gas: 217155)
[PASS] test_TokenNotApproved() (gas: 470170)
[PASS] test_ValidateRequestID() (gas: 880036)
[PASS] test_Version() (gas: 311094)
[PASS] test_ZeroTokenContract() (gas: 141411)
Test result: ok. 13 passed; 0 failed; finished in 3.63ms
```
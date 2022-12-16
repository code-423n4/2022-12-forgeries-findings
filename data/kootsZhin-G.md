## Summary
In total, `1` gas optimization is found.

## Optimiaztions
1. use `private constant` (or `constant`) for constants instead of `immutable` to save gas

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
| src/VRFNFTRandomDraw.sol:VRFNFTRandomDraw contract |                 |        |        |        |         |
|----------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 1237526                                            | 6717            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| contractVersion                                    | 272             | 272    | 272    | 272    | 1       |
| getRequestDetails                                  | 572             | 572    | 572    | 572    | 3       |
| hasUserWon                                         | 1764            | 1764   | 1764   | 1764   | 1       |
| initialize                                         | 43790           | 146546 | 175523 | 192923 | 15      |
| lastResortTimelockOwnerClaimNFT                    | 381             | 11061  | 11061  | 21741  | 2       |
| rawFulfillRandomWords                              | 1131            | 38496  | 46390  | 46390  | 6       |
| redraw                                             | 572             | 28944  | 28944  | 57317  | 2       |
| startDraw                                          | 482             | 86573  | 96467  | 99775  | 9       |
| winnerClaimNFT                                     | 422             | 11638  | 2422   | 27624  | 5       |
```

After (with `private constant`)
```
| src/VRFNFTRandomDraw.sol:VRFNFTRandomDraw contract |                 |        |        |        |         |
|----------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 1186680                                            | 6341            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| contractVersion                                    | 272             | 272    | 272    | 272    | 1       |
| getRequestDetails                                  | 572             | 572    | 572    | 572    | 3       |
| hasUserWon                                         | 1764            | 1764   | 1764   | 1764   | 1       |
| initialize                                         | 43790           | 146546 | 175523 | 192923 | 15      |
| lastResortTimelockOwnerClaimNFT                    | 381             | 11061  | 11061  | 21741  | 2       |
| rawFulfillRandomWords                              | 1131            | 38491  | 46384  | 46384  | 6       |
| redraw                                             | 572             | 28937  | 28937  | 57303  | 2       |
| startDraw                                          | 482             | 86557  | 96449  | 99757  | 9       |
| winnerClaimNFT                                     | 422             | 11638  | 2422   | 27624  | 5       |
```
## Gas 1: ```immutable``` variables that aren't declared in the constructor should be declared ```constant``` instead

https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L22-L33

```solidity
uint32 immutable callbackGasLimit = 200_000;
```
```solidity
uint16 immutable minimumRequestConfirmations = 3;
```
```solidity
uint16 immutable wordsRequested = 1;
```
```solidity
uint256 immutable HOUR_IN_SECONDS = 60 * 60;
```
```solidity
uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
```
```solidity
uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```
## Before:

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

## After:

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
## Wrap calculations in `unchecked` block if guaranteed not to over- or underflow
[VRFNFTRandomDraw.sol L112-L117](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L112-L117)
If statement with subtraction can be wrapped in `unchecked` because the statement already checks the first parameter shouldn't be smaller than the second. This slightly reduces contract size and function gas cost. Tests show the following gas cost reduction:
| src/VRFNFTRandomDraw.sol:VRFNFTRandomDraw contract |                 |        |        |        |         |
|----------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 1237526                                            | 6717            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| initialize                                         | 43790           | 146546 | 175523 | 192923 | 15      |
| **After update** |                 |        |        |        |         |
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 1235726                                            | 6708            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| initialize                                         | 43790           | 146503 | 175451 | 192851 | 15      |
| **DIFF** |                 |        |        |        |         |
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| **-1800**                                            | **-9**            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| initialize                                         | 0           | **-43** | **-72** | **-72** |       |

| src/VRFNFTRandomDrawFactory.sol:VRFNFTRandomDrawFactory contract |                 |        |        |        |         |
|------------------------------------------------------------------|-----------------|--------|--------|--------|---------|
| Function Name                                                    | min             | avg    | median | max    | # calls |
| makeNewDraw                                                      | 46872           | 183639 | 213232 | 239795 | 16      |
| **After update** |                 |        |        |        |         |
| makeNewDraw                                                      | 46872           | 183598 | 213160 | 239723 | 16      |
| **DIFF** |                 |        |        |        |         |
| makeNewDraw                                                      | 0           | **-41** | **-72** | **-72** |       |

## All event emitting can be moved to end of functions
The event emitting can be moved to the end of the functions. This save some gas in error cases and is in line with the checks-effects-interactions paradigm. There's some places where event emitting is done before checks.

## Redundant check in `startDraw()`
[VRFNFTRandomDraw.sol L175-L177](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L175-L177)
The check for `currentChainlinkRequestId` in `startDraw()` is redundant because the check is already present in `_requestRoll()`. Remove the check from `startDraw()` function and also move `emit SetupDraw(msg.sender, settings);` after `_requestRoll();` to save gas. Here's the benefits with these changes as observed from project tests:
| src/VRFNFTRandomDraw.sol:VRFNFTRandomDraw contract |                 |        |        |        |         |
|----------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 1237526                                            | 6717            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| startDraw                                          | 482             | 86573  | 96467  | 99775  | 9       |
| **After update** |                 |        |        |        |         |
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 1230919                                            | 6684            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| startDraw                                          | 497             | 86470  | 96350  | 99658  | 9       |
| **DIFF** |                 |        |        |        |         |
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| **-6607**                                            | **-33**            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| startDraw                                          | **+15**             | **-103**  | **-117**  | **-117**  | 0       |


## Storage variable can be cached in local variable
[VRFNFTRandomDraw.sol L250](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L250)
[VRFNFTRandomDraw.sol L256](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L256)
The storage variable `settings.drawingTokenStartId` is accessed twice inside `fulfillRandomWords()`. It can instead be cached in a local variable to save deployment and function gas cost. These are gas savings as observed with projects tests:

| src/VRFNFTRandomDraw.sol:VRFNFTRandomDraw contract |                 |        |        |        |         |
|----------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 1237526                                            | 6717            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| rawFulfillRandomWords                              | 1131            | 38496  | 46390  | 46390  | 6       |
| **After update** |                 |        |        |        |         |
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 1237126                                            | 6715            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| rawFulfillRandomWords                              | 1131            | 38415  | 46292  | 46292  | 6       |
| **DIFF** |                 |        |        |        |         |
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| **-400**                                            | **-2**            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| rawFulfillRandomWords                              | 0            | **-81**  | **-98**  | **-98**  |        |

## Redundant variable can be removed
[VRFNFTRandomDraw.sol L279](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L279)
The user local variable can be removed and msg.sender from calldata used instead. This slightly reduces contract size and function gas cost. Here's the project test results after removing the variable:

| src/VRFNFTRandomDraw.sol:VRFNFTRandomDraw contract |                 |        |        |        |         |
|----------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 1237526                                            | 6717            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| lastResortTimelockOwnerClaimNFT                    | 381             | 11061  | 11061  | 21741  | 2       |
| winnerClaimNFT                                     | 422             | 11638  | 2422   | 27624  | 5       |
| **After update** |                 |        |        |        |         |
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 1227313                                            | 6666            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| lastResortTimelockOwnerClaimNFT                    | 381             | 11061  | 11061  | 21742  | 2       |
| winnerClaimNFT                                     | 419             | 11635  | 2419   | 27621  | 5       |
| **DIFF** |                 |        |        |        |         |
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| **-10213**                                            | **-51**            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| lastResortTimelockOwnerClaimNFT                    | 0             | 0  | 0  | **+1**  |        |
| winnerClaimNFT                                     | **-3**             | **-3**  | **-3**   | **-3**  |        |

## Another redundant variable can be removed in `makeNewDraw()`
[VRFNFTRandomDrawFactory.sol L42](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L42)
The `admin` local variable can be removed and `msg.sender` used instead to save some gas. Here's the savings:
| src/VRFNFTRandomDrawFactory.sol:VRFNFTRandomDrawFactory contract |                 |        |        |        |         |
|------------------------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                                  | Deployment Size |        |        |        |         |
| 1042203                                                          | 5678            |        |        |        |         |
| Function Name                                                    | min             | avg    | median | max    | # calls |
| makeNewDraw                                                      | 46872           | 183639 | 213232 | 239795 | 16      |
| **After update** |                 |        |        |        |         |
| Deployment Cost                                                  | Deployment Size |        |        |        |         |
| 1041403                                                          | 5674            |        |        |        |         |
| Function Name                                                    | min             | avg    | median | max    | # calls |
| makeNewDraw                                                      | 46860           | 183631 | 213224 | 239783 | 16      |
| **DIFF** |                 |        |        |        |         |
| Deployment Cost                                                  | Deployment Size |        |        |        |         |
| **-800**                                                          | **-4**            |        |        |        |         |
| Function Name                                                    | min             | avg    | median | max    | # calls |
| makeNewDraw                                                      | **-12**           | **-8** | **-8** | **-12** |       |

## Use `msg.sender` instead of `owner()` function calls
[VRFNFTRandomDraw.solL312](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L312)
[VRFNFTRandomDraw.sol L317](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L317)
`lastResortTimelockOwnerClaimNFT()` already ensures that `msg.sender` is the owner with the `onlyOwner` modifier. Using the `owner()` function calls instead of using `msg.sender` is redundant in this case and increases gas cost. Here's the benefits after using `msg.sender` instead, as seen with the project tests:
| src/VRFNFTRandomDraw.sol:VRFNFTRandomDraw contract |                 |        |        |        |         |
|----------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 1237526                                            | 6717            |        |        |        |         |
| Function Name                                      | min             | avg    | median | max    | # calls |
| lastResortTimelockOwnerClaimNFT                    | 381             | 11061  | 11061  | 21741  | 2       |
| **After update** |                 |        |        |        |         |
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 1226513                                            | 6662            |        |        |        |         |
| lastResortTimelockOwnerClaimNFT                    | 381             | 10934  | 10934  | 21488  | 2       |
| **DIFF** |                 |        |        |        |         |
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| **-11013**                                            | **-55**            |        |        |        |         |
| lastResortTimelockOwnerClaimNFT                    | 0             | **-127**  | **-127**  | **-253**  |        |

## emitting events in `OwnableUpgradeable` can use `msg.sender`
[OwnableUpgradeable.sol L91](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L91)
[OwnableUpgradeable.sol L109](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L109)
[OwnableUpgradeable.sol L129](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L129)
Several events inside `OwnableUpgradeable` can use `msg.sender` as all calls are protected with `onlyOwner` modifier. This saves gas in several function calls:

| src/VRFNFTRandomDraw.sol:VRFNFTRandomDraw contract |                 |        |        |        |         |
|----------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 1237526                                            | 6717            |        |        |        |         |
| **After update** |                 |        |        |        |         |
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| 1231319                                            | 6686            |        |        |        |         |
| **DIFF** |                 |        |        |        |         |
| Deployment Cost                                    | Deployment Size |        |        |        |         |
| **-6207**                                            | **-31**            |        |        |        |         |

| src/VRFNFTRandomDrawFactory.sol:VRFNFTRandomDrawFactory contract |                 |        |        |        |         |
|------------------------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                                  | Deployment Size |        |        |        |         |
| 1042203                                                          | 5678            |        |        |        |         |
| Function Name                                                    | min             | avg    | median | max    | # calls |
| safeTransferOwnership                                            | 555             | 12443  | 12443  | 24331  | 2       |
| **After update** |                 |        |        |        |         |
| Deployment Cost                                                  | Deployment Size |        |        |        |         |
| 1035996                                                          | 5647            |        |        |        |         |
| Function Name                                                    | min             | avg    | median | max    | # calls |
| safeTransferOwnership                                            | 555             | 12379  | 12379  | 24204  | 2       |
| **DIFF** |                 |        |        |        |         |
| Deployment Cost                                                  | Deployment Size |        |        |        |         |
| **-6207**                                                          | **-31**            |        |        |        |         |
| Function Name                                                    | min             | avg    | median | max    | # calls |
| safeTransferOwnership                                            | 0             | **-64**  | **-64**  | **-127**  |        |

| src/VRFNFTRandomDrawFactoryProxy.sol:VRFNFTRandomDrawFactoryProxy contract |                 |       |        |       |         |
|----------------------------------------------------------------------------|-----------------|-------|--------|-------|---------|
| Function Name                                                              | min             | avg   | median | max   | # calls |
| safeTransferOwnership                                                      | 954             | 12840 | 12840  | 24726 | 2       |
| **After update** |                 |        |        |        |         |
| Function Name                                                              | min             | avg   | median | max   | # calls |
| safeTransferOwnership                                                      | 954             | 12776 | 12776  | 24599 | 2       |
| **DIFF** |                 |        |        |        |         |
| Function Name                                                              | min             | avg   | median | max   | # calls |
| safeTransferOwnership                                                      | 0             | **-64** | **-64**  | **-127** |        |


| test/OwnableTest.t.sol:OwnedContract contract |                 |       |        |       |         |
|-----------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                               | Deployment Size |       |        |       |         |
| 272664                                        | 1811            |       |        |       |         |
| Function Name                                 | min             | avg   | median | max   | # calls |
| cancelOwnershipTransfer                       | 1840            | 1840  | 1840   | 1840  | 1       |
| resignOwnership                               | 9239            | 9239  | 9239   | 9239  | 1       |
| safeTransferOwnership                         | 542             | 19374 | 25318  | 26318 | 4       |
| transferOwnership                             | 412             | 3454  | 4525   | 5426  | 3       |
| **After update** |                 |        |        |        |         |
| Deployment Cost                               | Deployment Size |       |        |       |         |
| 266458                                        | 1780            |       |        |       |         |
| cancelOwnershipTransfer                       | 1744            | 1744  | 1744   | 1744  | 1       |
| resignOwnership                               | 9121            | 9121  | 9121   | 9121  | 1       |
| safeTransferOwnership                         | 542             | 19278 | 25191  | 26191 | 4       |
| transferOwnership                             | 412             | 3383  | 4431   | 5308  | 3       |
| **DIFF** |                 |        |        |        |         |
| Deployment Cost                               | Deployment Size |       |        |       |         |
| **-6206**                                        | **-31**            |       |        |       |         |
| cancelOwnershipTransfer                       | 1744            | 1744  | 1744   | 1744  | 1       |
| resignOwnership                               | 9121            | 9121  | 9121   | 9121  | 1       |
| safeTransferOwnership                         | 542             | 19278 | 25191  | 26191 | 4       |
| transferOwnership                             | 412             | 3383  | 4431   | 5308  | 3       |

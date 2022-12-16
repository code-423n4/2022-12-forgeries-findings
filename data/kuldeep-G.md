# Case 1
VRFNFTRandomDraw. fulfillRandomWords: The checked mathematical operation can be unchecked.

## Explanation

`uint256 tokenRange = settings.drawingTokenEndId - settings.drawingTokenStartId;`

This above [calculation](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L249) can be unchecked as it has already been ensured in [initialize method](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L112) that this `_settings.drawingTokenEndId > _settings.drawingTokenStartId` condition will always hold true.

Gas usage:
                             | Function Name              | min   | avg    | median | max  | # calls |
Current logic => | rawFulfillRandomWords | 1131 | 38496 | 46390 | 46390 | 6 |
Suggested logic => | rawFulfillRandomWords | 1131 | 38354 | 46219 | 46219 | 6 |

# Case 2
IVRFNFTRandomDraw.Setting: Struct variable packing

## Explanation: 
In the `Settings` struct, the [subscriptionId](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L89) variable can be packed with `address token` or `address drawingToken` in the same storage slot to save ~20000 gas (one SSTORE will be removed).

Gas usage:
                   | Function Name    | min   | avg    | median | max  | # calls |
Current logic => | initialize  | 43790  | 146546 | 175523 | 192923 | 15 |
Suggested logic => | initialize | 41586 | 133672 | 153325 | 170725 | 15 |

# Case 3
VRFNFTRandomDraw. initialize: Redundant condition check in `if` statement

## Explanation

For the `VRFNFTRandomDraw` contract, this [check](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L113) in the initialize method, the first condition `_settings.drawingTokenEndId < _settings.drawingTokenStartId` is redundant as the second condition already ensures the same.

In case, `_settings.drawingTokenEndId < _settings.drawingTokenStartId`, the second check will result in `Arithmetic underflow` error automatically.

Gas usage:
With the current logic, the gas used is `test_BadDrawingRange() (gas: 277512)` whereas with the changes suggested above the gas used will be `test_BadDrawingRange() (gas: 277368)`.

note: if the suggested update is done in the contract then the associated test needs to be updated as well.

# Case 4
VRFNFTRandomDraw: Use solidity Time Units (hours, weeks) to save gas.

## Explanation
Remove [line 29](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L29) and [line 31](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L31). Use `1 hours` instead of `HOUR_IN_SECONDS` in [line 83](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L83). Use `1 weeks` instead of `WEEK_IN_SECONDS` in [line 90](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L90). This saves ~12k gas while deployment.

![Git diff](https://user-images.githubusercontent.com/24249646/208140023-c481d87b-cde7-453a-96e6-d99c2fd703fb.png)

Gas usage:
                            | Deployment cost | Deployment size
Current logic => | 1237526               | 6717
Suggested logic => | 1225659          | 6621
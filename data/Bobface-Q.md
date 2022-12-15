# Forgeries QA Report

The codebase is secure and well-structured. A few minor code-style issues and execution ordering errors are listed below.

# Low risk

## `redraw()` can be called before `startDraw()`
`VRFNFTRandomDraw.redraw()` can be successfully called before the first draw has been started through `startDraw()` by transferrig the NFT to the contract and then calling `redraw()`. This does not cause any substantial risk to the NFT, but it causes the contract to not emit the `SetupDraw` event when the first draw starts, potentially causing off-chain software tracking the events to have issues.

## `redraw()` does not emit an event
`VRFNFTRandomDraw.redraw()` does not emit an event, e.g. `Redraw`, making it hard for off-chain software to monitor for redraws. 

# Non-Critical

## Best Practices: Prefer `constant` over `immutable`
`constant` values are known at compile time and unchangeable. `immutable` values can be changed in the `constructor` and are unchangeable thereafter. If an `immutable` value is not changed in the `constructor`, the `constant` keyword should be used.
| File                    | Lines                         | Description |
|---|---|---|
| `src/VRFNFTRandomDraw.sol` | 22 | `callbackGasLimit` |
| `src/VRFNFTRandomDraw.sol` | 24 | `minimumRequestConfirmations` |
| `src/VRFNFTRandomDraw.sol` | 26 | `wordsRequested` |
| `src/VRFNFTRandomDraw.sol` | 29 | `HOUR_IN_SECONDS` |
| `src/VRFNFTRandomDraw.sol` | 31 | `WEEK_IN_SECONDS` |
| `src/VRFNFTRandomDraw.sol` | 33 | `MONTH_IN_SECONDS` |

## Best Practices: Specify state variable visibility
Visibility for state variables should be explicitly specified, even if the default `internal` is intended.
| File                    | Lines                         | Description |
|---|---|---|
| `src/VRFNFTRandomDraw.sol` | 22 | `callbackGasLimit` |
| `src/VRFNFTRandomDraw.sol` | 24 | `minimumRequestConfirmations` |
| `src/VRFNFTRandomDraw.sol` | 26 | `wordsRequested` |
| `src/VRFNFTRandomDraw.sol` | 29 | `HOUR_IN_SECONDS` |
| `src/VRFNFTRandomDraw.sol` | 31 | `WEEK_IN_SECONDS` |
| `src/VRFNFTRandomDraw.sol` | 33 | `MONTH_IN_SECONDS` |
| `src/VRFNFTRandomDraw.sol` | 37 | `coordinator` |
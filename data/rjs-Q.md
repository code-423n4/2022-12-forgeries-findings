# Introduction

The Forgeries codebase is very compact and exceedingly well structured and designed, leaving very little to comment on.

# Summary

| ID   | Finding                                                               |
| ---- | --------------------------------------------------------------------- |
| L-01 | `request.drawTimelock` is a fixed point in time                             |
| L-02 | `winnerClaimNFT()` does not handle the recovery scenario  |
| I-01 | Inconsistent parameter naming: `admin`                             |
| I-02 | Typo in comment: `winter`                             |

# Low Findings

## \[L-01\] `request.drawTimelock` is a fixed point in time

In `redraw()`, once `request.drawTimelock` has passed, it will be possible to keep requesting new rolls without any further delay.

An unethical owner might be able to exploit this in the event that the raffle winner does not claim fast enough, to keep rolling until a desired ticket wins, but there would be on-chain evidence of this behaviour so I don't believe this to be a significant finding.

## \[L-02\] `winnerClaimNFT()` does not handle the recovery scenario 

It is possible for the owner to invoke `lastResortTimelockOwnerClaimNFT()`, at which point the reward token will no longer be available to the contract.

`winnerClaimNFT()` does not check if this scenario has occurred, and nevertheless publishes an inaccurate `WinnerSentNFT` event because the subsequent code uses `transferFrom`, which will not revert in the event of an issue.

# Informational Findings

## \[I-01\] Inconsistent parameter naming: `admin`

I would expect this parameter to be named `_admin` to follow the convention used throughout the rest of the codebase;

```solidity
75:    function initialize(address admin, Settings memory _settings)
```

## \[I-02\] Typo in comment: `winter`

This should state `winner`, not `winter`:

```solidity
294:        // Transfer token to the winter.
```

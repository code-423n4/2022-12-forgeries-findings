# Introduction

The Forgeries codebase is very compact and exceedingly well structured and designed, leaving very little to comment on.

# Summary

| ID   | Finding                                                               |
| ---- | --------------------------------------------------------------------- |
| G-01 | Use cached variables                             |

# Gas Optimization Findings

## \[G-01\] Use cached variables

As per the preceding lines in the function `winnerClaimNFT`, `msg.sender` is already mapped to `user`:

```solidity
src\VRFNFTRandomDraw.sol
297:            msg.sender,
---

can be replaced with

```solidity
297:            user,
---
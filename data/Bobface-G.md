# Forgeries Gas Report

## Computed value should be stored as constant
A constant value is dynamically computed. It should be stored as constant to avoid recomputation.

| File                    | Lines                         | Description |
|---|---|---|
| `src/VRFNFTRandomDraw.sol` | 95 | `(MONTH_IN_SECONDS * 12)` can be stored as constant `YEAR_IN_SECONDS` |

## Redundant checks
A condition is unnecessarily checked multiple time. The duplicate check should be removed.
| File                    | Lines                         | Description |
|---|---|---|
| `src/VRFNFTRandomDraw.sol` | 126 - 137 | `initialize` checks that the owner of the contract owns the NFT to be raffled. However, it does not transfer in the NFT. The NFT is only transferred in in `startDraw`. The check in `initialize` can be removed, since if the NFT is not owned or does not exist, the check in `startDraw` will catch this condition. |
| `src/VRFNFTRandomDraw.sol` | 175 - 177 | Checking for `request.currentChainlinkRequestId != 0` is done again in `_requestRoll()`. |
| `src/utils/OwnableUpgradeable.sol` | 95 | The statement first reads (`SLOAD`) the storage slots, checks if for zero, and if it is non-zero, deletes (`SSTORE`) it. Removing the `if` guard and just always executing `delete _pendingOwner` is cheaper, since the overhead through the `if` condition is removed. |

## Storing `msg.sender` in a local variable
The EVM offers the `CALLER` opcode which only costs 2 gas (see [here](https://www.evm.codes/#33?fork=arrowGlacier)). `msg.sender` is directly translated to the `CALLER` opcode, which makes it more expensive to store `msg.sender` in a local stack variable due to the required stack operations. Using `msg.sender` directly where it is requied is cheaper.
| File                    | Lines                         | Description |
|---|---|---|
| `src/VRFNFTRandomDraw.sol` | 279 | `address user = msg.sender;` |
| `src/VRFNFTRandomDrawFactory.sol` | 42 | `address admin = msg.sender;` |

## Oversized types in storage structs
Using smaller types where possible in structs which are stored in storage decreases the type's size and thus requires less storage slots, making it cheaper to read and write the struct from and to storage.
| File                    | Lines                         | Description |
|---|---|---|
| `src/interfaces/IVRNFTRandomDraw.sol` | 67 | `drawTimelock` can be `uint128`, saving one storage slot |
| `src/interfaces/IVRNFTRandomDraw.sol` | 83, 85 | `drawBufferTime` and `recoverTimelock` can be `uint128`, saving one storage slot |
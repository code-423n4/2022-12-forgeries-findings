# [G-01] Using memory instead of storage variable when event emission saves gas

In `initialize` function inside `VRFNFTRandomDraw.sol` the event emission is using storage variable `settings`, it's best to use the memory variable `_settings`

```solidity
Before
File: VRFNFTRandomDraw.sol
122:         // Emit initialized event for indexing
123:         emit InitializedDraw(msg.sender, settings);

After
File: VRFNFTRandomDraw.sol
122:         // Emit initialized event for indexing
123:         emit InitializedDraw(msg.sender, _settings);
```

Gas differences

```markdown
| Function Name                                      | min             | avg    | median | max    | # calls |

Before
| initialize                                         | 43790           | 146546 | 175523 | 192923 | 15      |

After
| initialize                                         | 43790           | 146047 | 174692 | 192092 | 15      |
```

The average gas saving is `146546` - `146047` = `499`
Median `175523-174692 = 831`
Max `192923-192092 = 831`

# [G-02] Contract `Version` can be replaced with an immutable constant VERSION inside the contract

The `Version` contract doesn't have any benefit except returning information on what version is the contract. 

To save gas, we can remove this implementation, and directly create an immutable constant VERSION inside the contract itself, no need to create a separate contract. If needed, make it public so it can be retrieved if 

Remove:
```solidity
File: Version.sol
04: contract Version {
05:   uint32 private immutable __version;
06: 
07:   /// @notice The version of the contract
08:   /// @return The version ID of this contract implementation
09:   function contractVersion() external view returns (uint32) {
10:       return __version;
11:   }
12: 
13:   constructor(uint32 version) {
14:     __version = version;
15:   }
16: }
```
modify
```solidity
File: VRFNFTRandomDraw.sol
contract VRFNFTRandomDraw is
    IVRFNFTRandomDraw,
    VRFConsumerBaseV2,
    OwnableUpgradeable
{
    uint32 public immutable VERSION = 1
    ...
}
```
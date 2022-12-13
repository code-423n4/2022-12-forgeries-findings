# Use a locked pragma version
# Use same pragma version acrooss all contracts

# Unnecesary initializer modifier on constructor on `VRFNFTRandomDraw.sol`

`initializer` is not necessary at construction time since the constructor is being called just once.

## Where 
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L49

```solidity
    constructor(VRFCoordinatorV2Interface _coordinator)
        VRFConsumerBaseV2(address(_coordinator))
        // initializer - REMOVE THIS LINE
    {
        coordinator = _coordinator;
    }
```

# Low

# Calls to `resignOwnership` on `OwnableUpgradeable.sol` will lock contract's ownership forever

If the owner calls the function `resignOwnership` from an ownable contract, the ownership will be transfered to the zero address locking the ownership of that contract forever. The owner could call this function by mistake.

## Where
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/ownable/OwnableUpgradeable.sol#L114-L116

## Recommended solution
Remove this functionality or set a two step process instead.


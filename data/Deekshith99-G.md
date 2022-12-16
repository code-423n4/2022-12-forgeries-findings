 ### (GAS OPTIMIZATIONS)

## [G-1] State variables and structs can be closely packed to save gas

There are 2 instances of it  which can save 2 slots (20000 + 20000) can save total 40000 gas 

1st instance permalink is [here](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L29)
here instead of  `uint256 immutable HOUR_IN_SECONDS = 60 * 60;` 
can be written as `uint128 immutable HOUR_IN_SECONDS = 60 * 60;`
by this first 4  state variables  can fit in 1 storage slot only and can save 1 slot

2nd instance permalink is [here](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L89)
here in the setting struct  `uint64 subscriptionId;` (instead of placing at last ) 
if we place  `uint64 subscriptionId;` under the `address token;` then we save 1 slot 
because address - 20 , uint 64 - 8 can fit under 1 slot 


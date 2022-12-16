### Caching `msg.sender` uses more gas
There are two cases in the codebase where `msg.sender` is being stored into memory. Caching `msg.sender` uses more gass than using it directly. An `MSTORE` costs 3 gas and an `MLOAD` costs 3 gas, where as `CALLER` (`msg.sender`) costs just 2 gas.

### Recommendation
Remove the lines where `msg.sender` is being stored to memory and use `msg.sender` where you need the value instead.
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L279
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L42
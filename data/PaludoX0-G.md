## Admin caching increase gas consumption
src/VRFNFTRandomDrawFactory.sol#L42
By caching address `admin = msg.sender;` there's more gas consumption.
I've removed cache, change all admin occurances with msg.sender and tested using `test_FullDrawing()` and there is a saving of **800gas** in `VRFNFTRandomDrawFactory` deployment and **12gas** with `makeNewDraw()` call.


## Check of request.drawTimelock != 0 is useless
**VRFNFTRandomDraw.sol#L152**
In expression `if request.drawTimelock != 0 && request.drawTimelock > block.timestamp` in case `request.drawTimelock > block.timestamp` 
`request.drawTimelock` will be always grater than 0 so there's no need of `request.drawTimelock != 0` check


## Use memory variable in event emit
**VRFNFTRandomDraw.sol#L123**
You can use `_setting` memory variable instead of `setting` storage variable, this will save **831 gas** on calling `makeNewDraw()` function


## Cache storage variables
**VRFNFTRandomDraw.sol#L180 187 190**
You can cache `settings` storage variable 
`Settings memory _setting = settings` and substitute all occurance in fucntion.
This will save **84 gas** on calling `startDraw()` function.


## Move checking of drawing NFT ownership to VRFNFTRandomDrawFactory  
**VRFNFTRandomDraw.sol#L126**
Move checking of drawing NFT ownership to **src/VRFNFTRandomDrawFactory.sol#L41** in order to save gas in case the control is not satisfied.


## Constants instead of immutable if size is less than 32B
**VRFNFTRandomDraw.sol#L22 L24 and L26**
Declaring constants with size less than 32B should be sometimes cheaper than using immutable according to solidity docs.
This is stated in https://docs.soliditylang.org/en/v0.8.16/contracts.html#constant-and-immutable-state-variables


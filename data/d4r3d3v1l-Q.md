## Using Struct instead of constants saves more Gas

Code : 

```solidity

File : src/VRFNFTRandomDraw.sol

From Line 22

    uint32 immutable callbackGasLimit = 200_000;
    /// @notice Chainlink request confirmations, left at the default
    uint16 immutable minimumRequestConfirmations = 3;
    /// @notice Number of words requested in a drawing
    uint16 immutable wordsRequested = 1;

    /// @dev 60 seconds in a min, 60 mins in an hour
    uint256 immutable HOUR_IN_SECONDS = 60 * 60;
    /// @dev 24 hours in a day 7 days in a week
    uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
    // @dev about 30 days in a month
    uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;

```
[Link](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L22)

Here we have all constants , the gas cost of this storage is  `166482` 

If we use Packed struct with add appropriate datatype sizes , we can save gas 

**The packed struct look like this :**

```solidity

struct Vairables {
	/// As the value is 3 , we can use uint8  (requires 1 byte)
	uint8  minimumRequestConfirmations;
	/// Value is 1 so we can use uint8 (requries 1 byte)
	uint8  wordsRequested;
	///  Value is 3600 , which lies in uint16 (requires 2 bytes)
	uint16  HOUR_IN_SECONDS;
        /// Value is 200000 which lies in range of uint32
	uint32  callbackGasLimit;
        ///  Value is  604800 which lies in range of uint32
	uint32  WEEK_IN_SECONDS;
        /// Value is 18144000 which lies in range of uint32
	uint32  MONTH_IN_SECONDS;

}

Variables variables = Variables(3,1,3600,200000,604800,18144000);

```
This struct costs `134857` gas , which is less  than the previous one .
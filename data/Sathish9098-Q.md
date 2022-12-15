##

##[NC-1]  USE A MORE RECENT VERSION OF SOLIDITY

Use a solidity version of at least 0.8.4 to get bytes.concat() instead of abi.encodePacked(,) Use a solidity version of at least 0.8.12 to get string.concat() instead of abi.encodePacked(,) Use a solidity version of at least 0.8.13 to get the ability to use using for with a list of free functions

Version.sol :  pragma solidity ^0.8.10;

##

## [NC-2] Use same solidity versions for all contracts. Now contracts using the different solidity versions. Some contracts using stable version and some contracts using floating solidity versions. This is not a good code practice 

VRFNFTRandomDrawFactoryProxy.sol :  pragma solidity 0.8.16;

Version.sol :  pragma solidity ^0.8.10;






    









NC-1	Missing checks for address(0) when assigning values to address state variables	4
NC-2	Event is missing indexed fields	4
NC-3	Constants should be defined rather than using magic numbers	2
NC-4	Functions not used internally could be marked external	7
L-1	Unsafe ERC20 operation(s)	3
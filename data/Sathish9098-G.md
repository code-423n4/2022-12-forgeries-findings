##

## [GAS-1]  Use uint256 instead of uint32 . uint256 is consume less gas than uint32   

A smart contract's gas consumption can be higher if developers use items that are less than 32 bytes in size because the Ethereum Virtual Machine can only handle 32 bytes at a time. In order to increase the element's size to the necessary size, the EVM has to perform additional operations.

[File: 2022-12-forgeries/src/utils/Version.sol](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol)

      5:    uint32 private immutable __version;
   
     13:  constructor(uint32 version) {

[2022-12-forgeries/src/VRFNFTRandomDraw.sol](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol)

     22:  uint32 immutable callbackGasLimit = 200_000;

##

## [GAS-2] Use uint256 instead of uint16 . uint256 consume less gas than uint16   

[2022-12-forgeries/src/VRFNFTRandomDraw.sol](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol)

      24:   uint16 immutable minimumRequestConfirmations = 3;
   
      26:    uint16 immutable wordsRequested = 1;

##

## [GAS-3]  instead of using operator && on single require or if checks . using double REQUIRE OR IF checks can save more gas.

[2022-12-forgeries/src/VRFNFTRandomDraw.sol](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol)

          if (
            request.hasChosenRandomNumber &&
            // Draw timelock not yet used
            request.drawTimelock != 0 &&
            request.drawTimelock > block.timestamp
           )

##

## [GAS-4]  FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

[2022-12-forgeries/src/VRFNFTRandomDraw.sol](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol)

    
     203:  function redraw() external onlyOwner returns (uint256) {

     304 :  function lastResortTimelockOwnerClaimNFT() external onlyOwner {

##

## [ GAS-5]  






GAS-1	Use assembly to check for address(0)	3
GAS-2	Use calldata instead of memory for function arguments that do not get mutated	4
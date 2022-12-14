## [G-01] Use Shift Right/Left instead of Division/Multiplication if possible

A division/multiplication by any number x being a power of 2 can be calculated by shifting log2(x) to the right/left.

While the DIV opcode uses 5 gas, the SHR opcode only uses 3 gas. Furthermore, Solidity's division operation also includes a division-by-0 prevention which is bypassed using shifting.

```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::31 => uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
2022-12-forgeries/src/VRFNFTRandomDraw.sol::33 => uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```

## [G-02] Using private rather than public for constants, saves gas

If needed, the value can be read from the verified contract source code. Savings are due to the compiler not having to create non-payable getter functions for deployment calldata, and not adding another entry to the method ID table

```
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::21 => address public immutable implementation;
```

## [G-03] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::173 => function startDraw() external onlyOwner returns (uint256) {
2022-12-forgeries/src/VRFNFTRandomDraw.sol::203 => function redraw() external onlyOwner returns (uint256) {
2022-12-forgeries/src/VRFNFTRandomDraw.sol::304 => function lastResortTimelockOwnerClaimNFT() external onlyOwner {
```

## [G-04] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::22 => uint32 immutable callbackGasLimit = 200_000;
2022-12-forgeries/src/VRFNFTRandomDraw.sol::24 => uint16 immutable minimumRequestConfirmations = 3;
2022-12-forgeries/src/VRFNFTRandomDraw.sol::26 => uint16 immutable wordsRequested = 1;
2022-12-forgeries/src/utils/Version.sol::5 => uint32 private immutable __version;
```

# Gas Optimizations Report

|         | Issue                                                                            | Instances |
| ------- | :------------------------------------------------------------------------------- | :-------: |
| [G-001] | Functions guaranteed to revert when called by normal users can be marked payable |     6     |
| [G-002] | Replace modifier with function                                                   |     2     |
| [G-003] | Use named returns for local variables where it is possible.                      |     1     |

## [G-001] Functions guaranteed to revert when called by normal users can be marked payable

### Impact

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
The extra opcodes avoided are `CALLVALUE(2)`,`DUP1(3)`,`ISZERO(3)`,`PUSH2(3)`,`JUMPI(10)`,`PUSH1(3)`,`DUP1(3)`,`REVERT(0)`,`JUMPDEST(1)`,`POP(2)`, which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### Findings

Total:6

[src/VRFNFTRandomDraw.sol#L173](https://github.com/code-423n4/2022-12-forgeries/tree/main//src/VRFNFTRandomDraw.sol#L173)

```solidity
173:    function startDraw() external onlyOwner returns (uint256) {
```

[src/VRFNFTRandomDraw.sol#L304](https://github.com/code-423n4/2022-12-forgeries/tree/main//src/VRFNFTRandomDraw.sol#L304)

```solidity
304:    function lastResortTimelockOwnerClaimNFT() external onlyOwner {
```

[src/VRFNFTRandomDraw.sol#L203](https://github.com/code-423n4/2022-12-forgeries/tree/main//src/VRFNFTRandomDraw.sol#L203)

```solidity
203:    function redraw() external onlyOwner returns (uint256) {
```

[src/ownable/OwnableUpgradeable.sol#L114](https://github.com/code-423n4/2022-12-forgeries/tree/main//src/ownable/OwnableUpgradeable.sol#L114)

```solidity
114:    function resignOwnership() public onlyOwner {
```

[src/ownable/OwnableUpgradeable.sol#L119](https://github.com/code-423n4/2022-12-forgeries/tree/main//src/ownable/OwnableUpgradeable.sol#L119)

```solidity
119:    function acceptOwnership() public onlyPendingOwner {
```

[src/ownable/OwnableUpgradeable.sol#L128](https://github.com/code-423n4/2022-12-forgeries/tree/main//src/ownable/OwnableUpgradeable.sol#L128)

```solidity
128:    function cancelOwnershipTransfer() public onlyOwner {
```

## [G-002] Replace modifier with function

### Impact

Modifiers make code more elegant, but cost more than normal functions.

### Findings

Total:2

[src/ownable/OwnableUpgradeable.sol#L44](https://github.com/code-423n4/2022-12-forgeries/tree/main//src/ownable/OwnableUpgradeable.sol#L44)

```solidity
44:    modifier onlyPendingOwner() {
```

[src/ownable/OwnableUpgradeable.sol#L36](https://github.com/code-423n4/2022-12-forgeries/tree/main//src/ownable/OwnableUpgradeable.sol#L36)

```solidity
36:    modifier onlyOwner() {
```

## [G-003] Use named returns for local variables where it is possible.

### Impact

It is not necessary to have both a named return and a return statement.

### Findings

Total:1

[src/VRFNFTRandomDrawFactory.sol#L44](https://github.com/code-423n4/2022-12-forgeries/tree/main//src/VRFNFTRandomDrawFactory.sol#L44)

```solidity
44:    address newDrawing = ClonesUpgradeable.clone(implementation);
```

## Summary<a name="Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | Empty Blocks Should Be Removed Or Emit Something | 2 | - |
| [GAS&#x2011;2](#GAS&#x2011;2) | Public Functions To External | 6 | - |
| [GAS&#x2011;3](#GAS&#x2011;3) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 5 | - |
| [GAS&#x2011;4](#GAS&#x2011;4) | Optimize names to save gas | 3 | 66 |
| [GAS&#x2011;5](#GAS&#x2011;5) | Use `uint256(1)`/`uint256(2)` instead for `true` and `false` boolean states | 1 | 5000 |
| [GAS&#x2011;6](#GAS&#x2011;6) | `internal` functions only called once can be inlined to save gas | 1 | - |
| [GAS&#x2011;7](#GAS&#x2011;7) | Setting the `constructor` to `payable` | 4 | 52 |
| [GAS&#x2011;8](#GAS&#x2011;8) | Functions guaranteed to revert when called by normal users can be marked `payable` | 8 | 168 |

Total: 30 contexts over 8 issues

## Gas Optimizations

### <a href="#Summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> Empty Blocks Should Be Removed Or Emit Something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})

#### <ins>Proof Of Concept</ins>

```solidity
192: {} catch {
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L192

```solidity
59: {}
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDrawFactory.sol#L59







### <a href="#Summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Public Functions To External

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

#### <ins>Proof Of Concept</ins>


```solidity
function hasUserWon(address user) public view returns (bool) {
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L264

```solidity
function owner() public view virtual returns (address) {
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/ownable\OwnableUpgradeable.sol#L68

```solidity
function pendingOwner() public view returns (address) {
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/ownable\OwnableUpgradeable.sol#L73

```solidity
function resignOwnership() public onlyOwner {
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/ownable\OwnableUpgradeable.sol#L114

```solidity
function acceptOwnership() public onlyPendingOwner {
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/ownable\OwnableUpgradeable.sol#L119

```solidity
function cancelOwnershipTransfer() public onlyOwner {
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/ownable\OwnableUpgradeable.sol#L128






### <a href="#Summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

#### <ins>Proof Of Concept</ins>


```solidity
22: uint32 immutable callbackGasLimit = 200_000;
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L22

```solidity
24: uint16 immutable minimumRequestConfirmations = 3;
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L24

```solidity
26: uint16 immutable wordsRequested = 1;
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L26

```solidity
89: uint64 subscriptionId;
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/interfaces\IVRFNFTRandomDraw.sol#L89

```solidity
5: uint32 private immutable __version;
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/utils\Version.sol#L5






### <a href="#Summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

See more <a href="https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92">here</a>

#### <ins>Proof Of Concept</ins>

```solidity
File: .\Projects\forgeries\2022-12-forgeries\src\VRFNFTRandomDrawFactoryProxy.sol
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDrawFactoryProxy.sol

```solidity
File: .\Projects\forgeries\2022-12-forgeries\src\ownable\OwnableUpgradeable.sol
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/ownable\OwnableUpgradeable.sol

```solidity
File: .\Projects\forgeries\2022-12-forgeries\src\utils\Version.sol
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/utils\Version.sol



#### <ins>Recommended Mitigation Steps</ins>
Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas
For example, the function IDs in the Gauge.sol contract will be the most used; A lower method ID may be given.



### <a href="#Summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Use `uint256(1)`/`uint256(2)` instead for `true` and `false` boolean states

If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from `true` to `false` can cost up to ~20000 gas rather than `uint256(2)` to `uint256(1)` that would cost significantly less.

#### <ins>Proof Of Concept</ins>


```solidity
246: request.hasChosenRandomNumber = true;
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L246






### <a href="#Summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> `internal` functions only called once can be inlined to save gas

#### <ins>Proof Of Concept</ins>

```solidity
230: function fulfillRandomWords

```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L230






### <a href="#Summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Setting the `constructor` to `payable`

Saves ~13 gas per instance

#### <ins>Proof Of Concept</ins>

```solidity
47: constructor(VRFCoordinatorV2Interface _coordinator)
        VRFConsumerBaseV2(address(_coordinator))
        initializer
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L47

```solidity
24: constructor(address _implementation) initializer
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDrawFactory.sol#L24

```solidity
12: constructor(address _logic, address _defaultOwner)
        ERC1967Proxy(
            _logic,
            abi.encodeWithSelector(
                IVRFNFTRandomDrawFactory.initialize.selector,
                _defaultOwner
            )
        )
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDrawFactoryProxy.sol#L12

```solidity
13: constructor(uint32 version)
```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/utils\Version.sol#L13





### <a href="#Summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### <ins>Proof Of Concept</ins>

```solidity
173: function startDraw() external onlyOwner returns (uint256) {

```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L173

```solidity
203: function redraw() external onlyOwner returns (uint256) {

```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L203

```solidity
304: function lastResortTimelockOwnerClaimNFT() external onlyOwner {

```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/VRFNFTRandomDraw.sol#L304

```solidity
79: function transferOwnership(address _newOwner)
        public
        notZeroAddress(_newOwner)
        onlyOwner
    {

```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/ownable\OwnableUpgradeable.sol#L79

```solidity
102: function safeTransferOwnership(address _newOwner)
        public
        notZeroAddress(_newOwner)
        onlyOwner
    {

```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/ownable\OwnableUpgradeable.sol#L102

```solidity
114: function resignOwnership() public onlyOwner {

```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/ownable\OwnableUpgradeable.sol#L114

```solidity
119: function acceptOwnership() public onlyPendingOwner {

```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/ownable\OwnableUpgradeable.sol#L119

```solidity
128: function cancelOwnershipTransfer() public onlyOwner {

```

https://github.com/code-423n4/2022-12-forgeries/tree/main/src/ownable\OwnableUpgradeable.sol#L128



#### <ins>Recommended Mitigation Steps</ins>
Functions guaranteed to revert when called by normal users can be marked payable.




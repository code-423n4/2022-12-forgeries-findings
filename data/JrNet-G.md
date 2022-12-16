## [G001] >= costs less gas than >

he compiler uses opcodes `GT` and `ISZERO` for solidity code that uses `>`, but only requires `LT` for `>=`, which saves 3 gas

#### Instances:
```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::86 => if (_settings.drawBufferTime > MONTH_IN_SECONDS) {
2022-12-forgeries/src/VRFNFTRandomDraw.sol::204 => if (request.drawTimelock >= block.timestamp) {
2022-12-forgeries/src/VRFNFTRandomDraw.sol::306 => if (settings.recoverTimelock > block.timestamp) {
```
## [G002] Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.\[layout_in_storage.html]{https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html}\Use a larger size then downcast where needed

#### Instances:
```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::22 => uint32 immutable callbackGasLimit = 200_000;
2022-12-forgeries/src/VRFNFTRandomDraw.sol::24 => uint16 immutable minimumRequestConfirmations = 3;
2022-12-forgeries/src/VRFNFTRandomDraw.sol::26 => uint16 immutable wordsRequested = 1;
```
## [G003] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (`if(x){}else if(y){...}else{...}` => `if(!x){if(y){...}else{...}}`)

#### Instances:
```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::192 => {} catch {
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::59 => {}
2022-12-forgeries/src/VRFNFTRandomDrawFactoryProxy.sol::20 => {}
```
## [G004] Functions with access control cheaper if payable

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are `CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

#### Instances:
```
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::114 => function resignOwnership() public onlyOwner {
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::119 => function acceptOwnership() public onlyPendingOwner {
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::128 => function cancelOwnershipTransfer() public onlyOwner {
```
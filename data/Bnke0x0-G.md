

### [G01] Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead


#### Findings:
```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::22 => uint32 immutable callbackGasLimit = 200_000;
2022-12-forgeries/src/VRFNFTRandomDraw.sol::24 => uint16 immutable minimumRequestConfirmations = 3;
2022-12-forgeries/src/VRFNFTRandomDraw.sol::26 => uint16 immutable wordsRequested = 1;
2022-12-forgeries/src/interfaces/IVRFNFTRandomDraw.sol::89 => uint64 subscriptionId;
2022-12-forgeries/src/utils/Version.sol::5 => uint32 private immutable __version;
2022-12-forgeries/src/utils/Version.sol::9 => function contractVersion() external view returns (uint32) {
2022-12-forgeries/src/utils/Version.sol::13 => constructor(uint32 version) {
```



### [G02] Duplicated `require()`/`revert()` checks should be refactored to a modifier or function


#### Findings:
```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::133 => revert DOES_NOT_OWN_NFT();
2022-12-forgeries/src/VRFNFTRandomDraw.sol::145 => revert REQUEST_IN_FLIGHT();
```





### [G03] Functions guaranteed to revert when called by normal users can be marked `payable`

#### Impact
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
#### Findings:
```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::173 => function startDraw() external onlyOwner returns (uint256) {
2022-12-forgeries/src/VRFNFTRandomDraw.sol::203 => function redraw() external onlyOwner returns (uint256) {
2022-12-forgeries/src/VRFNFTRandomDraw.sol::304 => function lastResortTimelockOwnerClaimNFT() external onlyOwner {
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::114 => function resignOwnership() public onlyOwner {
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::119 => function acceptOwnership() public onlyPendingOwner {
2022-12-forgeries/src/ownable/OwnableUpgradeable.sol::128 => function cancelOwnershipTransfer() public onlyOwner {
```






### [G04] Empty blocks should be removed or emit something

#### Impact
The code should be refactored such that they no longer exist, or the 
block should do something useful, such as emitting an event or 
reverting. If the contract is meant to be extended, the contract should 
be abstract and the function signatures be added without 
any default implementation. If the block is an empty if-statement block 
to avoid doing subsequent checks in the else-if/else conditions, the 
else-if/else conditions should be nested under the negation of the 
if-statement, because they involve different classes of checks, which 
may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})
#### Findings:
```
2022-12-forgeries/src/VRFNFTRandomDraw.sol::192 => {} catch {
2022-12-forgeries/src/VRFNFTRandomDrawFactory.sol::59 => {}
2022-12-forgeries/src/VRFNFTRandomDrawFactoryProxy.sol::20 => {}
```


# GAS
## Empty blocks should be removed or emit something
### Impact
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

### Github Permalinks
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDrawFactory.sol#L59
    {}

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L192
        {} catch {


### Mitigation
Remove empty block, revert or emit something.


## Unnecesary storage read
### Summary
A value which value is already known can be used directly rather than reading it from the storage
### Example
```
    // Set new settings
    settings = _settings; 

    ...

     emit InitializedDraw(msg.sender, settings);
```

Recommendation
Change to:

```
    // Set new settings
    settings = _settings; 

    ...

     emit InitializedDraw(msg.sender, _settings); //@audit can be used value known
```

### Github Permalinks
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L123
        emit InitializedDraw(msg.sender, settings);

### Mitigation
Set directly the value to avoid unnecessary storage read to save some gas


## Variables should be cached when used several times
### Summary
Variables read more than once improves gas usage when cached into local variable
### Details
In state variables, this is even more gas saving

For example saving as
`IVRFNFTRandomDraw.CurrentRequest memory localVar = request;`
Where accessed multiple times storage for reading 

### Github Permalinks
`owner()`
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L312-L317
`settings`
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L306-L318
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L288-L298
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L249-L256
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L180-L191
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L158-L164
`request`
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L235-L259
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L203-L225
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L144-L153
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L67-L69

### Mitigation
Cache variables used more than one into a local variable.








### Mitigation
Consider changing internal function only called once to inline code for gas savings


## Pack structs tightly
### Summary
Gas efficiency can be achieved by tightly packing the struct. Struct variables are stored in 32 bytes each so you can group smaller types to occupy less storage. 

### Details
You can read more here: https://fravoll.github.io/solidity-patterns/tight_variable_packing.html or in the official documentation: https://docs.soliditylang.org/en/v0.4.21/miscellaneous.html
### Github Permalinks
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/interfaces/IVRFNFTRandomDraw.sol#L71
### Mitigation
Order structs to reduce gas usage.

## Functions guaranteed to revert when called by normal users can be marked `payable`
### Summary
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function.

Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

### Details
The extra opcodes avoided are:
CALLVALUE (2), DUP1 (3), ISZERO (3), PUSH2 (3), JUMPI (10), PUSH1 (3), DUP1 (3), REVERT(0), JUMPDEST (1), POP (2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

### Github Permalinks

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDrawFactory.sol#L58
        onlyOwner

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L173
    function startDraw() external onlyOwner returns (uint256) {

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L203
    function redraw() external onlyOwner returns (uint256) {

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L304
    function lastResortTimelockOwnerClaimNFT() external onlyOwner {

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/ownable/OwnableUpgradeable.sol#L82
        onlyOwner

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/ownable/OwnableUpgradeable.sol#L105
        onlyOwner

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/ownable/OwnableUpgradeable.sol#L114
    function resignOwnership() public onlyOwner {

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/ownable/OwnableUpgradeable.sol#L128
    function cancelOwnershipTransfer() public onlyOwner {



### Mitigation
Consider adding payable to save gas

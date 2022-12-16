## Missing checks can make `_requestRoll` fail
### Summary
Zero address should be checked. In case not checked and wrongly assigned, a redeploy would be needed as `_requestRoll` would fail when called
### Github Permalinks
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L51
```
    constructor(VRFCoordinatorV2Interface _coordinator)
        VRFConsumerBaseV2(address(_coordinator))
        initializer
    {
        coordinator = _coordinator;//@audit not checked here, neither on VRFConsumerBaseV2 
    }
```

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L162-L168
```
     request.currentChainlinkRequestId = coordinator.requestRandomWords({
            keyHash: settings.keyHash,
            subId: settings.subscriptionId,
            minimumRequestConfirmations: minimumRequestConfirmations,
            callbackGasLimit: callbackGasLimit,
            numWords: wordsRequested
        });
```
### Mitigation
Check zero address in the constructor

## Risky operations at `OwnableUpgradeable`
### Impact
Critical address transfer/resign in one-step is very risky because it is irrecoverable from any mistakes. 

Even if the contract contains a safer version of transfer, both are able to be used, therefore, safer version can be ignored.

Given that `VRFNFTRandomDrawFactory` and ``VRFNFTRandomDraw` are derived from `OwnableUpgradeable`, the ownership management of this contract defaults to `OwnableUpgradeable` ’s `transferOwnership()` and `resignOwnership()` methods which are not overridden here. 

Therefore functions from both contracts that rely on `onlyOwner` can get unable to be used anymore

### Github Permalinks
- `Ownable`
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDrawFactory.sol#L16
    OwnableUpgradeable,

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L18
    OwnableUpgradeable,

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/ownable/OwnableUpgradeable.sol#L12
abstract contract OwnableUpgradeable is IOwnableUpgradeable, Initializable {

### Recommended steps
Recommend overriding the inherited methods to null functions and use the two step function

Also, consider adding a time-delay for such sensitive actions.



## Upgradeable contract is missing a __gap[50] storage variable to allow for new storage variables in later versions
### Impact 
For upgradeable contracts, there must be storage gap to "allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments" (quote OpenZeppelin). Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts, potentially causing loss of user fund or cause the contract to malfunction completely.

See:
https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
For a description of this storage variable. While some contracts may not currently be sub-classed, adding the variable now protects against forgetting to add it in the future.

### Proof of Concept
In the following context of the upgradeable contracts they are expected to use gaps for avoiding collision:
- `UUPSUpgradeable`
- `OwnableUpgradeable`

However, none of these contracts contain storage gap. The storage gap is essential for upgradeable contract because "It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments". Refer to the bottom part of this article:

https://docs.openzeppelin.com/contracts/4.x/upgradeable

If a contract inheriting from a base contract contains additional variable, then the base contract cannot be upgraded to include any additional variable, because it would overwrite the variable declared in its child contract. This greatly limits contract upgradeability.

### Github Permalinks
`OwnableUpgradeable` and `UUPSUpgradeable`
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDrawFactory.sol#L16
    OwnableUpgradeable,

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDrawFactory.sol#L17
    UUPSUpgradeable,

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L18
    OwnableUpgradeable, //@audit upgradeable

### Tools Used
Manual analysis

### Recommended Mitigation Steps
Recommend adding appropriate storage gap at the end of upgradeable contracts such as the below. 
Please reference OpenZeppelin upgradeable contract templates.

`uint256[50] private __gap;`








## `block.timestamp` used as time proxy 
### Summary
Risk of using `block.timestamp` for time should be considered. 
### Details
`block.timestamp` is not an ideal proxy for time because of issues with synchronization, miner manipulation and changing block times. 

This kind of issue may affect the code allowing or reverting the code before the expected deadline, modifying the normal functioning or reverting sometimes.
### References
SWC ID: 116

### Github Permalinks 
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L90
        if (_settings.recoverTimelock < block.timestamp + WEEK_IN_SECONDS) {

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L95
            block.timestamp + (MONTH_IN_SECONDS * 12)

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L153
            request.drawTimelock > block.timestamp

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L159
        request.drawTimelock = block.timestamp + settings.drawBufferTime;

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L204
        if (request.drawTimelock >= block.timestamp) {

https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDraw.sol#L306
        if (settings.recoverTimelock > block.timestamp) {



### Mitigation
- Consider the risk of using `block.timestamp` as time proxy and evaluate if block numbers can be used as an approximation for the application logic. Both have risks that need to be factored in. 
- Consider using an oracle for precision


## Front run initializer
### Summary
The initialize function that initializes important contract state can be called by anyone.
### Details
The attacker can initialize the contract before the legitimate deployer, hoping that the victim continues to use the same contract.

In the best case for the victim, they notice it and have to redeploy their contract costing gas.

### Github Permalinks
https://github.com/code-423n4/2022-12-forgeries/blob/01e6cbed2896246a4635708e1aeee247526872a1/src/VRFNFTRandomDrawFactory.sol#L31
    function initialize(address _initialOwner) initializer external {

### Mitigation
Use the constructor to initialize non-proxied contracts.

For initializing proxy contracts deploy contracts using a factory contract that immediately calls initialize after deployment or make sure to call it immediately after deployment and verify the transaction succeeded.




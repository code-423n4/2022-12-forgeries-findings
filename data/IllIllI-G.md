

## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Structs can be packed into fewer storage slots | 1 |  - |
| [G&#x2011;02] | State variables should be cached in stack variables rather than re-reading them from storage | 3 |  291 |
| [G&#x2011;03] | Optimize names to save gas | 7 |  154 |
| [G&#x2011;04] | Functions guaranteed to revert when called by normal users can be marked `payable` | 10 |  210 |

Total: 21 instances over 4 issues with **655 gas** saved

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations

### [G&#x2011;01]  Structs can be packed into fewer storage slots
Each slot saved can avoid an extra Gsset (**20000 gas**) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings

*There is 1 instance of this issue:*

```solidity
File: src/interfaces/IVRFNFTRandomDraw.sol

/// @audit Variable ordering with 8 slots instead of the current 9:
///           uint256(32):tokenId, uint256(32):drawingTokenStartId, uint256(32):drawingTokenEndId, uint256(32):drawBufferTime, uint256(32):recoverTimelock, bytes32(32):keyHash, address(20):token, uint64(8):subscriptionId, address(20):drawingToken
71        struct Settings {
72            /// @notice Token Contract to put up for raffle
73            address token;
74            /// @notice Token ID to put up for raffle
75            uint256 tokenId;
76            /// @notice Token that each (sequential) ID has a entry in the raffle.
77            address drawingToken;
78            /// @notice Start token ID for the drawing (if totalSupply = 20 but the first token is 5 (5-25), setting this to 5 would fix the ordering)
79            uint256 drawingTokenStartId;
80            /// @notice End token ID for the drawing (exclusive) (token ids 0 - 9 would be 10 in this field)
81            uint256 drawingTokenEndId;
82            /// @notice Draw buffer time â€“ time until a re-drawing can occur if the selected user cannot or does not claim the NFT.
83            uint256 drawBufferTime;
84            /// @notice block.timestamp that the admin can recover the NFT (as a safety fallback)
85            uint256 recoverTimelock;
86            /// @notice Chainlink gas keyhash
87            bytes32 keyHash;
88            /// @notice Chainlink subscription id
89            uint64 subscriptionId;
90:       }

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L71-L90

### [G&#x2011;02]  State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (**100 gas**) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

*There are 3 instances of this issue:*

```solidity
File: src/ownable/OwnableUpgradeable.sol

/// @audit _pendingOwner on line 95
96:               delete _pendingOwner;

/// @audit _pendingOwner on line 122
124:          delete _pendingOwner;

/// @audit _pendingOwner on line 129
131:          delete _pendingOwner;

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/ownable/OwnableUpgradeable.sol#L96

### [G&#x2011;03]  Optimize names to save gas
`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save **128 gas** each during deployment, and renaming functions to have lower method IDs will save **22 gas** per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

*There are 7 instances of this issue:*

```solidity
File: src/interfaces/IVRFNFTRandomDrawFactory.sol

/// @audit initialize(), makeNewDraw()
6:    interface IVRFNFTRandomDrawFactory {

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDrawFactory.sol#L6

```solidity
File: src/interfaces/IVRFNFTRandomDraw.sol

/// @audit initialize(), startDraw(), redraw(), hasUserWon(), winnerClaimNFT(), lastResortTimelockOwnerClaimNFT(), getRequestDetails()
4:    interface IVRFNFTRandomDraw {

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L4

```solidity
File: src/ownable/IOwnableUpgradeable.sol

/// @audit pendingOwner(), safeTransferOwnership(), cancelOwnershipTransfer()
7:    interface IOwnableUpgradeable {

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/ownable/IOwnableUpgradeable.sol#L7

```solidity
File: src/ownable/OwnableUpgradeable.sol

/// @audit pendingOwner(), safeTransferOwnership(), resignOwnership(), cancelOwnershipTransfer()
12:   abstract contract OwnableUpgradeable is IOwnableUpgradeable, Initializable {

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/ownable/OwnableUpgradeable.sol#L12

```solidity
File: src/utils/Version.sol

/// @audit contractVersion()
4:    contract Version {

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/utils/Version.sol#L4

```solidity
File: src/VRFNFTRandomDrawFactory.sol

/// @audit initialize(), makeNewDraw()
14:   contract VRFNFTRandomDrawFactory is

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L14

```solidity
File: src/VRFNFTRandomDraw.sol

/// @audit getRequestDetails(), initialize(), startDraw(), redraw(), hasUserWon(), winnerClaimNFT(), lastResortTimelockOwnerClaimNFT()
15:   contract VRFNFTRandomDraw is

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L15

### [G&#x2011;04]  Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are 
`CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about **21 gas per call** to the function, in addition to the extra deployment cost

*There are 10 instances of this issue:*

```solidity
File: src/ownable/OwnableUpgradeable.sol

57        function __Ownable_init(address _initialOwner)
58            internal
59            notZeroAddress(_initialOwner)
60:           onlyInitializing

79        function transferOwnership(address _newOwner)
80            public
81            notZeroAddress(_newOwner)
82:           onlyOwner

102       function safeTransferOwnership(address _newOwner)
103           public
104           notZeroAddress(_newOwner)
105:          onlyOwner

114:      function resignOwnership() public onlyOwner {

119:      function acceptOwnership() public onlyPendingOwner {

128:      function cancelOwnershipTransfer() public onlyOwner {

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/ownable/OwnableUpgradeable.sol#L57-L60

```solidity
File: src/VRFNFTRandomDrawFactory.sol

55        function _authorizeUpgrade(address newImplementation)
56            internal
57            override
58:           onlyOwner

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L55-L58

```solidity
File: src/VRFNFTRandomDraw.sol

173:      function startDraw() external onlyOwner returns (uint256) {

203:      function redraw() external onlyOwner returns (uint256) {

304:      function lastResortTimelockOwnerClaimNFT() external onlyOwner {

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L173


___

## Excluded findings
These findings are excluded from awards calculations because there are publicly-available automated tools that find them. The valid ones appear here for completeness

## Summary

### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| [G&#x2011;01] | Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas | 2 |  240 |

Total: 2 instances over 1 issues with **240 gas** saved

Gas totals use lower bounds of ranges and count two iterations of each `for`-loop. All values above are runtime, not deployment, values; deployment values are listed in the individual issue descriptions. The table above as well as its gas numbers do not include any of the excluded findings.





## Gas Optimizations

### [G&#x2011;01]  Using `calldata` instead of `memory` for read-only arguments in `external` functions saves gas
When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the `calldata` to the `memory` index. **Each iteration of this for-loop costs at least 60 gas** (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having `memory` arguments, it's still valid for implementation contracs to use `calldata` arguments instead. 

If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it's still more gass-efficient to use `calldata` when the `external` function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I've also flagged instances where the function is `public` but can be marked as `external` since it's not called by the contract, and cases where a constructor is involved

*There are 2 instances of this issue:*

```solidity
File: src/VRFNFTRandomDrawFactory.sol

/// @audit settings - (valid but excluded finding)
38        function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)
39            external
40:           returns (address)

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L38-L40

```solidity
File: src/VRFNFTRandomDraw.sol

/// @audit _settings - (valid but excluded finding)
75        function initialize(address admin, Settings memory _settings)
76            public
77:           initializer

```
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L75-L77

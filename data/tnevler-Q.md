# Report
## Non-Critical Issues ##
### [N-1]: Wrong order of functions
**Context:**

1. ```constructor(uint32 version) {``` [L13](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol#L13) (constructor can not go after external function)
1. ```function startDraw() external onlyOwner returns (uint256) {``` [L173](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L173) (external function can not go after internal function)
1. ```function owner() public view virtual returns (address) {``` [L68](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol#L68) (public function can not go after internal function)

**Description:**

According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) functions should be grouped according to their visibility and ordered:

+ constructor

+ receive function (if exists)

+ fallback function (if exists)

+ external

+ public

+ internal

+ private

Within a grouping, place the view and pure functions last.

**Recommendation:**

Put the functions in the correct order according to the documentation.

### [N-2]: NatSpec is missing
**Context:**

1. ```function initialize(address _initialOwner) initializer external {``` [L31](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L31) 

### [N-3]: Typos
**Context:**

1. ```/// @notice When the owner reclaims nft aftr recovery time delay``` [L46](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L46) (Change ***aftr*** to ***after***)

### [N-4]: Line is too long
**Context:**

1. ```/// @notice has chosen a random number (in case random number = 0(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0))``` [L64](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L64) 
1. ```/// @notice Start token ID for the drawing (if totalSupply = 20 but the first token is 5 (5-25), setting this to 5 would fix the ordering)``` [L78](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L78) 
1. ```/// @notice Draw buffer time – time until a re-drawing can occur if the selected user cannot or does not claim the NFT.``` [L82](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L82) 

**Description:**

Maximum suggested line length is [120 characters](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length).

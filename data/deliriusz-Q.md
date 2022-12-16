## [L-01] `VRFNFTRandomDrawFactory` initialized conditionally in deployment script
https://github.com/code-423n4/2022-12-forgeries/blob/main/script/SetupContractsScript.s.sol#L35-L42
[Deployment script](https://github.com/code-423n4/2022-12-forgeries/blob/main/script/SetupContractsScript.s.sol) only initializes `VRFNFTRandomDrawFactory` when `proxyNewOwner` is not `0`, which will lead to uninitialized contract and anyone could initialize the contract and become it's owner. If however it's meant to always be deployed with newAddress, then, deploying `VRFNFTRandomDrawFactoryProxy` every time is (1) gas intensive, (2) contradicts a reason to use upgradeable proxy in the first place (OZ `ERC1967Proxy` is upgradeable)

## [L-02] `VRFNFTRandomDraw` does not support IERC721Receiver interface
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L15
Because the smart contracts keeps ERC721s in an escrow, it should implement
https://docs.openzeppelin.com/contracts/2.x/api/token/erc721#IERC721Receiver - "Interface for any contract that wants to support safe transfers from ERC721 asset contracts." While it should be used for `safeTransferFrom` function which is not used in the smart contract, some NFTs may not comply to the spec and require it.

## [L-03] No storage gap for upgradeable contract
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L22
Upgradeable contracts should have storage `__gap`  . Even though right now nothing inherits from the smart contract, it may in the future.

Please refer to https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps for details.

## [L-04] `VRFNFTRandomDraw::initialize` does not check if admin address is not **0**
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L75

Upgrading to a new version of `VRFNFTRandomDraw` could brick the smart contract.

## [L-05] Parameter name mismatch in `VRFNFTRandomDraw::_requestRoll()` 
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L165

A function parameter `minimumRequestConfirmations` in `requestRandomWords` function is named incorrectly.
```
minimumRequestConfirmations: minimumRequestConfirmations,
```
In `VRFCoordinatorV2.requestConfirmations`  function it's called `requestConfirmations` and should be changed, as it will always have default 3 blocks reqeust confirmations.

```
  function requestRandomWords(
    bytes32 keyHash,
    uint64 subId,
    uint16 requestConfirmations,
    uint32 callbackGasLimit,
    uint32 numWords
  ) external override nonReentrant returns (uint256) {
```


## [QA-01] Constant variables should use `constant` keyword
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L22-L33

VRFNFTRandomDraw implements constants variables marked as `immutable`. Because they are not changed, should be marked as `constant` to make it clear that they are not settable on deployment.

## [QA-02] Time constants can be replaced with Solidity's keywords
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L28-L33
Solidity provides specific keywords for period, like `days`, `weeks`, `months`, those are easier to read and reason about than calculating time manually.
```
    /// @dev 60 seconds in a min, 60 mins in an hour
    uint256 immutable HOUR_IN_SECONDS = 60 * 60;
    /// @dev 24 hours in a day 7 days in a week
    uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
    // @dev about 30 days in a month
    uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```

## [QA-03] Unclear comment line
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L148
Comment in this line is unclear, and should be changed
```
// If the number has been drawn and
```
## [QA-01] `VRFCoordinatorV2` is not used and can be deleted
```
import {VRFCoordinatorV2, VRFCoordinatorV2Interface} from "@chainlink/contracts/src/v0.8/VRFCoordinatorV2.sol";
```

## [QA-04] According to Chainlink's documentation, `fulfillRandomWords` should not revert
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L236
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L242

Please refer to the documentation concerning this issue https://docs.chain.link/vrf/v2/security#fulfillrandomwords-must-not-revert

```
        // Validate request ID
        if (_requestId != request.currentChainlinkRequestId) {
            revert REQUEST_DOES_NOT_MATCH_CURRENT_ID();
        }

        // Validate number of words returned
        // Words requested is an immutable set to 1
        if (_randomWords.length != wordsRequested) {
            revert WRONG_LENGTH_FOR_RANDOM_WORDS();
        }
```

## [QA-05] Typo in comment
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L294
```
// Transfer token to the winter.
```

## [QA-06] Unclear variable name in `winnerClaimNFT`
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L279

*msg.sender* is cached into `user` variable at the beginning of `winnerClaimNFT() function`. The varabile name may be misleading, it could be named `caller` or `msgSender` for better readibility.
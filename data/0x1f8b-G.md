- [Gas](#gas)
    - [**1. Reorder structure layout**](#1-reorder-structure-layout)
        - [forge test --gas-report](#forge-test---gas-report)
    - [**2. Don't cache msg.sender**](#2-dont-cache-msgsender)
        - [forge test --gas-report](#forge-test---gas-report)
    - [**3. Use solidity literals instead of math**](#3-use-solidity-literals-instead-of-math)
        - [forge test --gas-report](#forge-test---gas-report)
    - [**4. Optimize VRFNFTRandomDraw.initialize**](#4-optimize-vrfnftrandomdrawinitialize)
        - [forge test --gas-report](#forge-test---gas-report)
    - [**5. Avoid duplicate logics**](#5-avoid-duplicate-logics)
        - [forge test --gas-report](#forge-test---gas-report)
    - [**6. Use constant instead of immutable**](#6-use-constant-instead-of-immutable)
        - [forge test --gas-report](#forge-test---gas-report)

# Gas

## **1. Reorder structure layout**

The following structures could be optimized moving the position of certain values in order to save some storage slots:

`Settings` in [IVRFNFTRandomDraw.sol:71-91](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/interfaces/IVRFNFTRandomDraw.sol#L71-L91):

```diff
    struct Settings {
        /// @notice Token Contract to put up for raffle
        address token;
+       uint64 subscriptionId;
        /// @notice Token ID to put up for raffle
        uint256 tokenId;
        /// @notice Token that each (sequential) ID has a entry in the raffle.
        address drawingToken;
        /// @notice Start token ID for the drawing (if totalSupply = 20 but the first token is 5 (5-25), setting this to 5 would fix the ordering)
        uint256 drawingTokenStartId;
        /// @notice End token ID for the drawing (exclusive) (token ids 0 - 9 would be 10 in this field)
        uint256 drawingTokenEndId;
        /// @notice Draw buffer time – time until a re-drawing can occur if the selected user cannot or does not claim the NFT.
        uint256 drawBufferTime;
        /// @notice block.timestamp that the admin can recover the NFT (as a safety fallback)
        uint256 recoverTimelock;
        /// @notice Chainlink gas keyhash
        bytes32 keyHash;
        /// @notice Chainlink subscription id
-       uint64 subscriptionId;
    }
```

**Reference:**

- https://ethdebug.github.io/solidity-data-representation
- https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html#storage-inplace-encoding

### forge test --gas-report

"before" in red, "after" in green. In general the deployment size size is larger, but the execution is cheaper.

```diff
    | src/VRFNFTRandomDraw.sol:VRFNFTRandomDraw contract |                 |        |        |        |         |
    |----------------------------------------------------|-----------------|--------|--------|--------|---------|
    | Deployment Cost                                    | Deployment Size |        |        |        |         |
-   | 1237526                                            | 6717            |        |        |        |         |
+   | 1238926                                            | 6724            |        |        |        |         |
    | Function Name                                      | min             | avg    | median | max    | # calls |
-   | getRequestDetails                                  | 572             | 572    | 572    | 572    | 3       |
+   | getRequestDetails                                  | 550             | 550    | 550    | 550    | 3       |
-   | initialize                                         | 43790           | 146546 | 175523 | 192923 | 15      |
+   | initialize                                         | 41586           | 133672 | 153325 | 170725 | 15      |
-   | rawFulfillRandomWords                              | 1131            | 38496  | 46390  | 46390  | 6       |
+   | rawFulfillRandomWords                              | 1109            | 38474  | 46368  | 46368  | 6       |
-   | redraw                                             | 572             | 28944  | 28944  | 57317  | 2       |
+   | redraw                                             | 572             | 28951  | 28951  | 57331  | 2       |
-   | startDraw                                          | 482             | 86573  | 96467  | 99775  | 9       |
+   | startDraw                                          | 549             | 86569  | 96454  | 99762  | 9       |
-   | winnerClaimNFT                                     | 422             | 11638  | 2422   | 27624  | 5       |
+   | winnerClaimNFT                                     | 422             | 11606  | 2422   | 27546  | 5       |

    | src/VRFNFTRandomDrawFactory.sol:VRFNFTRandomDrawFactory contract |                 |        |        |        |         |
    |------------------------------------------------------------------|-----------------|--------|--------|--------|---------|
    | Deployment Cost                                                  | Deployment Size |        |        |        |         |
-   | 1042203                                                          | 5678            |        |        |        |         |
+   | 1044210                                                          | 5688            |        |        |        |         |
    | Function Name                                                    | min             | avg    | median | max    | # calls |
-   | acceptOwnership                                                  | 1966            | 1966   | 1966   | 1966   | 1       |
+   | acceptOwnership                                                  | 2019            | 2019   | 2019   | 2019   | 1       |
-   | contractVersion                                                  | 250             | 250    | 250    | 250    | 1       |
+   | contractVersion                                                  | 228             | 228    | 228    | 228    | 1       |
-   | initialize                                                       | 881             | 20519  | 27065  | 27065  | 4       |
+   | initialize                                                       | 926             | 20564  | 27110  | 27110  | 4       |
-   | makeNewDraw                                                      | 46872           | 183639 | 213232 | 239795 | 16      |
+   | makeNewDraw                                                      | 46944           | 171637 | 200269 | 217669 | 16      |
-   | owner                                                            | 376             | 376    | 376    | 376    | 3       |
+   | owner                                                            | 365             | 365    | 365    | 365    | 3       |
-   | proxiableUUID                                                    | 352             | 352    | 352    | 352    | 1       |
+   | proxiableUUID                                                    | 319             | 319    | 319    | 319    | 1       |
-   | safeTransferOwnership                                            | 555             | 12443  | 12443  | 24331  | 2       |
+   | safeTransferOwnership                                            | 600             | 12488  | 12488  | 24376  | 2       |
-   | upgradeTo                                                        | 845             | 4078   | 5555   | 5834   | 3       |
+   | upgradeTo                                                        | 823             | 4045   | 5500   | 5812   | 3       |

    | src/VRFNFTRandomDrawFactoryProxy.sol:VRFNFTRandomDrawFactoryProxy contract |                 |       |        |       |         |
    |----------------------------------------------------------------------------|-----------------|-------|--------|-------|---------|
    | Deployment Cost                                                            | Deployment Size |       |        |       |         |
-   | 192212                                                                     | 1850            |       |        |       |         |
+   | 192257                                                                     | 1850            |       |        |       |         |
    | Function Name                                                              | min             | avg   | median | max   | # calls |
-   | acceptOwnership                                                            | 2280            | 2280  | 2280   | 2280  | 1       |
+   | acceptOwnership                                                            | 2332            | 2332  | 2332   | 2332  | 1       |
-   | makeNewDraw                                                                | 47315           | 47315 | 47315  | 47315 | 1       |
+   | makeNewDraw                                                                | 47387           | 47387 | 47387  | 47387 | 1       |
-   | owner                                                                      | 771             | 771   | 771    | 771   | 2       |
+   | owner                                                                      | 760             | 760   | 760    | 760   | 2       |
-   | safeTransferOwnership                                                      | 954             | 12840 | 12840  | 24726 | 2       |
+   | safeTransferOwnership                                                      | 999             | 12885 | 12885  | 24771 | 2       |
-   | upgradeTo                                                                  | 1244            | 4474  | 5950   | 6230  | 3       |
+   | upgradeTo                                                                  | 1222            | 4441  | 5895   | 6208  | 3       |
```

## **2. Don't cache `msg.sender`**

```diff
    function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)
        external
        returns (address)
    {
-       address admin = msg.sender;
        // Clone the contract
        address newDrawing = ClonesUpgradeable.clone(implementation);
        // Setup the new drawing
-       IVRFNFTRandomDraw(newDrawing).initialize(admin, settings);
+       IVRFNFTRandomDraw(newDrawing).initialize(msg.sender, settings);
        // Emit event for indexing
-       emit SetupNewDrawing(admin, newDrawing);
+       emit SetupNewDrawing(msg.sender, newDrawing);
        // Return address for integration or testing
        return newDrawing;
    }
```

**Affected source code:**

- [VRFNFTRandomDrawFactory.sol:38-51](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L38-L51)
- [VRFNFTRandomDraw.sol:279](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L279)

### forge test --gas-report

No negative impact was found, "before" in red, "after" in green

```diff
    | src/VRFNFTRandomDraw.sol:VRFNFTRandomDraw contract |                 |        |        |        |         |
    |----------------------------------------------------|-----------------|--------|--------|--------|---------|
    | Deployment Cost                                    | Deployment Size |        |        |        |         |
-   | 1237526                                            | 6717            |        |        |        |         |
+   | 1227313                                            | 6666            |        |        |        |         |
    | Function Name                                      | min             | avg    | median | max    | # calls |
-   | winnerClaimNFT                                     | 422             | 11638  | 2422   | 27624  | 5       |
+   | winnerClaimNFT                                     | 419             | 11635  | 2419   | 27621  | 5       |

    | src/VRFNFTRandomDrawFactory.sol:VRFNFTRandomDrawFactory contract |                 |        |        |        |         |
    |------------------------------------------------------------------|-----------------|--------|--------|--------|---------|
    | Deployment Cost                                                  | Deployment Size |        |        |        |         |
-   | 1042203                                                          | 5678            |        |        |        |         |
+   | 1041403                                                          | 5674            |        |        |        |         |
    | Function Name                                                    | min             | avg    | median | max    | # calls |
-   | makeNewDraw                                                      | 46872           | 183639 | 213232 | 239795 | 16      |
+   | makeNewDraw                                                      | 46860           | 183631 | 213224 | 239783 | 16      |

    | src/VRFNFTRandomDrawFactoryProxy.sol:VRFNFTRandomDrawFactoryProxy contract |                 |       |        |       |         |
    |----------------------------------------------------------------------------|-----------------|-------|--------|-------|---------|
    | Function Name                                                              | min             | avg   | median | max   | # calls |
-   | makeNewDraw                                                                | 47315           | 47315 | 47315  | 47315 | 1       |
+   | makeNewDraw                                                                | 47303           | 47303 | 47303  | 47303 | 1       |
```

## **3. Use solidity literals instead of math**

Suffixes like `seconds`, `minutes`, `hours`, `days` and `weeks` after literal numbers can be used to specify units of time where seconds are the base unit and units are considered naively in the following way:

- 1 == 1 seconds
- 1 minutes == 60 seconds
- 1 hours == 60 minutes
- 1 days == 24 hours
- 1 weeks == 7 days

Avoiding the `immutable` variables we will have a cheaper deployment.

**Reference:**

- https://docs.soliditylang.org/en/v0.8.14/units-and-global-variables.html#time-units

```js
   uint256 immutable HOUR_IN_SECONDS = 60 * 60;                 => 1 hours;
   uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);         => 1 weeks;
   uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;   => 30 days;
```

Also it can be optimized the use of `MONTH_IN_SECONDS` in:

```diff
        if (
            _settings.recoverTimelock >
-           block.timestamp + (MONTH_IN_SECONDS * 12)
+           block.timestamp + 365 days
        ) {
            revert RECOVER_TIMELOCK_NEEDS_TO_BE_LESS_THAN_A_YEAR();
        }
```

**Affected source code:**

- [VRFNFTRandomDraw.sol:28-33](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L28-L33)
- [VRFNFTRandomDraw.sol:95](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L95)

### forge test --gas-report

No negative impact was found, "before" in red, "after" in green

```diff
    | src/VRFNFTRandomDraw.sol:VRFNFTRandomDraw contract |                 |        |        |        |         |
    |----------------------------------------------------|-----------------|--------|--------|--------|---------|
    | Deployment Cost                                    | Deployment Size |        |        |        |         |
-   | 1237526                                            | 6717            |        |        |        |         |
+   | 1205796                                            | 6495            |        |        |        |         |
    | Function Name                                      | min             | avg    | median | max    | # calls |
-   | initialize                                         | 43790           | 146546 | 175523 | 192923 | 15      |
+   | initialize                                         | 43790           | 146472 | 175431 | 192831 | 15      |

    | src/VRFNFTRandomDrawFactory.sol:VRFNFTRandomDrawFactory contract |                 |        |        |        |         |
    |------------------------------------------------------------------|-----------------|--------|--------|--------|---------|
    | Function Name                                                    | min             | avg    | median | max    | # calls |
-   | makeNewDraw                                                      | 46872           | 183639 | 213232 | 239795 | 16      |
+   | makeNewDraw                                                      | 46872           | 183570 | 213140 | 239703 | 16      |
```

## **4. Optimize `VRFNFTRandomDraw.initialize`**

Avoid setting variables before using it, it will improve error executions.

```diff
    /// @notice Initialize the contract with settings and an admin
    /// @param admin initial admin user
    /// @param _settings initial settings for draw
    function initialize(address admin, Settings memory _settings)
        public
        initializer
    {
-       // Set new settings
-       settings = _settings;

        // Check values in memory:
        if (_settings.drawBufferTime < HOUR_IN_SECONDS) {
            revert REDRAW_TIMELOCK_NEEDS_TO_BE_MORE_THAN_AN_HOUR();
        }
        if (_settings.drawBufferTime > MONTH_IN_SECONDS) {
            revert REDRAW_TIMELOCK_NEEDS_TO_BE_LESS_THAN_A_MONTH();
        }

        if (_settings.recoverTimelock < block.timestamp + WEEK_IN_SECONDS) {
            revert RECOVER_TIMELOCK_NEEDS_TO_BE_AT_LEAST_A_WEEK();
        }
        if (
            _settings.recoverTimelock >
            block.timestamp + (MONTH_IN_SECONDS * 12)
        ) {
            revert RECOVER_TIMELOCK_NEEDS_TO_BE_LESS_THAN_A_YEAR();
        }

        // If NFT contract address is not a contract
        if (_settings.token.code.length == 0) {
            revert TOKEN_NEEDS_TO_BE_A_CONTRACT(_settings.token);
        }

        // If drawing token is not a contract
        if (_settings.drawingToken.code.length == 0) {
            revert TOKEN_NEEDS_TO_BE_A_CONTRACT(_settings.drawingToken);
        }

        // Validate token range: end needs to be greater than start
        // and the size of the range needs to be at least 2 (end is exclusive)
        if (
            _settings.drawingTokenEndId < _settings.drawingTokenStartId ||
            _settings.drawingTokenEndId - _settings.drawingTokenStartId < 2
        ) {
            revert DRAWING_TOKEN_RANGE_INVALID();
        }

+       // Set new settings
+       settings = _settings;

        // Setup owner as admin
        __Ownable_init(admin);

        ...
    }
```

**Affected source code:**

- [VRFNFTRandomDraw.sol:80](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L80)

### forge test --gas-report

"before" in red, "after" in green.

```diff
    | src/VRFNFTRandomDraw.sol:VRFNFTRandomDraw contract |                 |        |        |        |         |
    |----------------------------------------------------|-----------------|--------|--------|--------|---------|
    | Deployment Cost                                    | Deployment Size |        |        |        |         |
-   | 1237526                                            | 6717            |        |        |        |         |
+   | 1238126                                            | 6720            |        |        |        |         |
    | Function Name                                      | min             | avg    | median | max    | # calls |
-   | initialize                                         | 43790           | 146546 | 175523 | 192923 | 15      |
+   | initialize                                         | 23732           | 121280 | 175529 | 192929 | 15      |

    | src/VRFNFTRandomDrawFactory.sol:VRFNFTRandomDrawFactory contract |                 |        |        |        |         |
    |------------------------------------------------------------------|-----------------|--------|--------|--------|---------|
    | Function Name                                                    | min             | avg    | median | max    | # calls |
-   | makeNewDraw                                                      | 46872           | 183639 | 213232 | 239795 | 16      |
+   | makeNewDraw                                                      | 46872           | 159952 | 213238 | 239801 | 16      |
```

## **5. Avoid duplicate logics**

The following check it will be done during the `_requestRoll` call, so it can be removed.

```diff
    /// @notice Call this to start the raffle drawing
    /// @return chainlink request id
    function startDraw() external onlyOwner returns (uint256) {
-       // Only can be called on first drawing
-       if (request.currentChainlinkRequestId != 0) {
-           revert REQUEST_IN_FLIGHT();
-       }

        // Emit setup draw user event
        emit SetupDraw(msg.sender, settings);

        // Request initial roll
        _requestRoll();
```

**Affected source code:**

- [VRFNFTRandomDraw.sol:175](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L175)

### forge test --gas-report

"before" in red, "after" in green

> Note that the increase of `Deployment Size` in `startDraw` must be a forge error, because it will remove code and logic, so it can't be more than before.

```diff
    | src/VRFNFTRandomDraw.sol:VRFNFTRandomDraw contract |                 |        |        |        |         |
    |----------------------------------------------------|-----------------|--------|--------|--------|---------|
    | Deployment Cost                                    | Deployment Size |        |        |        |         |
-   | 1237526                                            | 6717            |        |        |        |         |
+   | 1230919                                            | 6684            |        |        |        |         |
    | Function Name                                      | min             | avg    | median | max    | # calls |
-   | startDraw                                          | 482             | 86573  | 96467  | 99775  | 9       |
+   | startDraw                                          | 5152            | 86985  | 96347  | 99655  | 9       |
```

## **6. Use `constant` instead of `immutable`**

If the value won't be changed during the constructor, it's better to use a constant instead of an `immutable` because it's cheaper.

```diff
    /// @notice Our callback is just setting a few variables, 200k should be more than enough gas.
-   uint32 immutable callbackGasLimit = 200_000;
+   uint32 constant callbackGasLimit = 200_000;
    /// @notice Chainlink request confirmations, left at the default
-   uint16 immutable minimumRequestConfirmations = 3;
+   uint16 constant minimumRequestConfirmations = 3;
    /// @notice Number of words requested in a drawing
-   uint16 immutable wordsRequested = 1;
+   uint16 constant wordsRequested = 1;

    /// @dev 60 seconds in a min, 60 mins in an hour
-   uint256 immutable HOUR_IN_SECONDS = 60 * 60;
+   uint256 constant HOUR_IN_SECONDS = 60 * 60;
    /// @dev 24 hours in a day 7 days in a week
-   uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
+   uint256 constant WEEK_IN_SECONDS = (3600 * 24 * 7);
    // @dev about 30 days in a month
-   uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
+   uint256 constant MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```

**Affected source code:**

- [VRFNFTRandomDraw.sol:21-33](https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L21-L33)

### forge test --gas-report

No negative impact was found, "before" in red, "after" in green

```diff
    | src/VRFNFTRandomDraw.sol:VRFNFTRandomDraw contract |                 |        |        |        |         |
    |----------------------------------------------------|-----------------|--------|--------|--------|---------|
    | Deployment Cost                                    | Deployment Size |        |        |        |         |
-   | 1237526                                            | 6717            |        |        |        |         |
+   | 1186680                                            | 6341            |        |        |        |         |
    | Function Name                                      | min             | avg    | median | max    | # calls |
-   | rawFulfillRandomWords                              | 1131            | 38496  | 46390  | 46390  | 6       |
+   | rawFulfillRandomWords                              | 1131            | 38491  | 46384  | 46384  | 6       |
-   | redraw                                             | 572             | 28944  | 28944  | 57317  | 2       |
+   | redraw                                             | 572             | 28937  | 28937  | 57303  | 2       |
-   | startDraw                                          | 482             | 86573  | 96467  | 99775  | 9       |
+   | startDraw                                          | 482             | 86557  | 96449  | 99757  | 9       |
```



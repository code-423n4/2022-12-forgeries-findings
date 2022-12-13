# Remove auxiliar variable admin on `VRFNFTRandomDrawFactory.sol`

## Where
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L38-L51


```solidity
  function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)
      external
      returns (address)
  {
      address admin = msg.sender;
      // Clone the contract
      address newDrawing = ClonesUpgradeable.clone(implementation);
      // Setup the new drawing
      IVRFNFTRandomDraw(newDrawing).initialize(admin, settings);
      // Emit event for indexing
      emit SetupNewDrawing(admin, newDrawing);
      // Return address for integration or testing
      return newDrawing;
  }
```

Change it with

```solidity
  function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)
      external
      returns (address)
  {
      // Clone the contract
      address newDrawing = ClonesUpgradeable.clone(implementation);
      // Setup the new drawing
      IVRFNFTRandomDraw(newDrawing).initialize(msg.sender, settings);
      // Emit event for indexing
      emit SetupNewDrawing(msg.sender, newDrawing);
      // Return address for integration or testing
      return newDrawing;
  }
```

# Replace auxiliar variable `newDrawing` with a return variable `newDrawing`

```solidity
  function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)
      external
      returns (address)
  {
      address admin = msg.sender;
      // Clone the contract
      address newDrawing = ClonesUpgradeable.clone(implementation);
      // Setup the new drawing
      IVRFNFTRandomDraw(newDrawing).initialize(admin, settings);
      // Emit event for indexing
      emit SetupNewDrawing(admin, newDrawing);
      // Return address for integration or testing
      return newDrawing;
  }
```

Replace it with

```solidity
  function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)
      external
      returns (address newDrawing)
  {
      address admin = msg.sender;
      // Clone the contract
      newDrawing = ClonesUpgradeable.clone(implementation);
      // Setup the new drawing
      IVRFNFTRandomDraw(newDrawing).initialize(admin, settings);
      // Emit event for indexing
      emit SetupNewDrawing(admin, newDrawing);
  }
```

# Use `calldata` instead of `memory` to save gas on function input parameters

## Where
* https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L75
* https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDrawFactory.sol#L38

```solidity
    function initialize(address admin, Settings memory _settings)
        public
        initializer
    {
```

```solidity
    function makeNewDraw(IVRFNFTRandomDraw.Settings memory settings)
        external
        returns (address)
    {
```
 
Replace it with

```solidity
    function initialize(address admin, Settings calldata _settings)
        public
        initializer
    {
```

```solidity
    function makeNewDraw(IVRFNFTRandomDraw.Settings calldata settings)
        external
        returns (address)
    {
```

# Move the `settings` initialization after the validations to save gas if any validation is not fullfilled

## Where
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L80

```solidity
  function initialize(address admin, Settings memory _settings)
      public
      initializer
  {
      // Set new settings
      settings = _settings;

      // ... _settings VALIDATIONS
```

Change it with:

```solidity
  function initialize(address admin, Settings memory _settings)
      public
      initializer
  {
      // ... _settings VALIDATIONS

      // Set new settings
      settings = _settings;
```

# Remove auxiliar variable `user`

## Where
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L277-L300

```solidity
    function winnerClaimNFT() external {
        // Assume (potential) winner calls this fn, cache.
        address user = msg.sender;

        // Check if this user has indeed won.
        if (!hasUserWon(user)) {
            revert USER_HAS_NOT_WON();
        }

        // Emit a celebratory event
        emit WinnerSentNFT(
            user,
            address(settings.token),
            settings.tokenId,
            settings
        );

        // Transfer token to the winter.
        IERC721EnumerableUpgradeable(settings.token).transferFrom(
            address(this),
            msg.sender,
            settings.tokenId
        );
    }
```

Replace it with:

```solidity
    function winnerClaimNFT() external {
        // Check if this user has indeed won.
        if (!hasUserWon(msg.sender)) {
            revert USER_HAS_NOT_WON();
        }

        // Emit a celebratory event
        emit WinnerSentNFT(
            msg.sender,
            address(settings.token),
            settings.tokenId,
            settings
        );

        // Transfer token to the winter.
        IERC721EnumerableUpgradeable(settings.token).transferFrom(
            address(this),
            msg.sender,
            settings.tokenId
        );
    }
```

# Cache `settings` in memory

## Where
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L277-L300

```solidity
    function winnerClaimNFT() external {
        // Assume (potential) winner calls this fn, cache.
        address user = msg.sender;

        // Check if this user has indeed won.
        if (!hasUserWon(user)) {
            revert USER_HAS_NOT_WON();
        }

        // Emit a celebratory event
        emit WinnerSentNFT(
            user,
            address(settings.token),
            settings.tokenId,
            settings
        );

        // Transfer token to the winter.
        IERC721EnumerableUpgradeable(settings.token).transferFrom(
            address(this),
            msg.sender,
            settings.tokenId
        );
    }
```

Replace it with 

```solidity
    function winnerClaimNFT() external {
        // Assume (potential) winner calls this fn, cache.
        address user = msg.sender;

        Settings memory settings_ = settings;

        // Check if this user has indeed won.
        if (!hasUserWon(user)) {
            revert USER_HAS_NOT_WON();
        }

        // Emit a celebratory event
        emit WinnerSentNFT(
            user,
            address(settings_.token),
            settings_.tokenId,
            settings_
        );

        // Transfer token to the winter.
        IERC721EnumerableUpgradeable(settings_.token).transferFrom(
            address(this),
            msg.sender,
            settings_.tokenId
        );
    }
```

# Use `constant` instead of `immutable` if such variables won't be reassigned on construction tiem

## Where
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L22-L33

```solidity
    /// @notice Our callback is just setting a few variables, 200k should be more than enough gas.
    uint32 immutable callbackGasLimit = 200_000;
    /// @notice Chainlink request confirmations, left at the default
    uint16 immutable minimumRequestConfirmations = 3;
    /// @notice Number of words requested in a drawing
    uint16 immutable wordsRequested = 1;

    /// @dev 60 seconds in a min, 60 mins in an hour
    uint256 immutable HOUR_IN_SECONDS = 60 * 60;
    /// @dev 24 hours in a day 7 days in a week
    uint256 immutable WEEK_IN_SECONDS = (3600 * 24 * 7);
    // @dev about 30 days in a month
    uint256 immutable MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```

Replace it with

```solidity
    /// @notice Our callback is just setting a few variables, 200k should be more than enough gas.
    uint32 constant callbackGasLimit = 200_000;
    /// @notice Chainlink request confirmations, left at the default
    uint16 constant minimumRequestConfirmations = 3;
    /// @notice Number of words requested in a drawing
    uint16 constant wordsRequested = 1;

    /// @dev 60 seconds in a min, 60 mins in an hour
    uint256 constant HOUR_IN_SECONDS = 60 * 60;
    /// @dev 24 hours in a day 7 days in a week
    uint256 constant WEEK_IN_SECONDS = (3600 * 24 * 7);
    // @dev about 30 days in a month
    uint256 constant MONTH_IN_SECONDS = (3600 * 24 * 7) * 30;
```

# Move `_requestRoll()` call below the nft transfer call

## Where
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L183-L194

```solidity
    // Request initial roll
    _requestRoll();

    // Attempt to transfer token into this address
    try
        IERC721EnumerableUpgradeable(settings.token).transferFrom(
            msg.sender,
            address(this),
            settings.tokenId
        )
    {} catch {
        revert TOKEN_NEEDS_TO_BE_APPROVED_TO_CONTRACT();
    }
```

Replace it with

```solidity
    // Attempt to transfer token into this address
    try
        IERC721EnumerableUpgradeable(settings.token).transferFrom(
            msg.sender,
            address(this),
            settings.tokenId
        )
    {} catch {
        revert TOKEN_NEEDS_TO_BE_APPROVED_TO_CONTRACT();
    }

    // Request initial roll
    _requestRoll();
```

# Move the event trigger below the nft transfer

## Where
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L179-L194

```solidity
    // Emit setup draw user event
    emit SetupDraw(msg.sender, settings);

    // Request initial roll
    _requestRoll();

    // Attempt to transfer token into this address
    try
        IERC721EnumerableUpgradeable(settings.token).transferFrom(
            msg.sender,
            address(this),
            settings.tokenId
        )
    {} catch {
        revert TOKEN_NEEDS_TO_BE_APPROVED_TO_CONTRACT();
    }
```

Replace it with

```solidity
    // Request initial roll
    _requestRoll();

    // Attempt to transfer token into this address
    try
        IERC721EnumerableUpgradeable(settings.token).transferFrom(
            msg.sender,
            address(this),
            settings.tokenId
        )
    {} catch {
        revert TOKEN_NEEDS_TO_BE_APPROVED_TO_CONTRACT();
    }

    // Emit setup draw user event
    emit SetupDraw(msg.sender, settings);
```

# Move nft ownership validation above the storage initialization on `initialize` call

## Where
https://github.com/code-423n4/2022-12-forgeries/blob/fc271cf20c05ce857d967728edfb368c58881d85/src/VRFNFTRandomDraw.sol#L119-L137

```solidity
    // Set new settings
    settings = _settings;

    // ...

    // Setup owner as admin
    __Ownable_init(admin);

    // Emit initialized event for indexing
    emit InitializedDraw(msg.sender, settings);

    // Get owner of raffled tokenId and ensure the current owner is the admin
    try
        IERC721EnumerableUpgradeable(_settings.token).ownerOf(
            _settings.tokenId
        )
    returns (address nftOwner) {
        // Check if address is the admin address
        if (nftOwner != admin) {
            revert DOES_NOT_OWN_NFT();
        }
    } catch {
        revert TOKEN_BEING_OFFERED_NEEDS_TO_EXIST();
    }
```

Move the try catch block above the storage initialization and event trigger

```solidity
    // Get owner of raffled tokenId and ensure the current owner is the admin
    try
        IERC721EnumerableUpgradeable(_settings.token).ownerOf(
            _settings.tokenId
        )
    returns (address nftOwner) {
        // Check if address is the admin address
        if (nftOwner != admin) {
            revert DOES_NOT_OWN_NFT();
        }
    } catch {
        revert TOKEN_BEING_OFFERED_NEEDS_TO_EXIST();
    }

    // Set new settings
        settings = _settings;

    // Setup owner as admin
    __Ownable_init(admin);

    // Emit initialized event for indexing
    emit InitializedDraw(msg.sender, settings);
```
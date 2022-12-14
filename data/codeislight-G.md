- in makeNewDraw function, newDrawing variable could be defined as a variable in the return area to save some gas

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
- use unchecked in calculations that is impossible to overflow:

        function _requestRoll() internal {
           ...
            unchecked{
                // Setup re-draw timelock
                request.drawTimelock = block.timestamp + settings.drawBufferTime;
            }
            ...
        }

        function fulfillRandomWords(
            uint256 _requestId,
            uint256[] memory _randomWords
        ) internal override {
            ...
            unchecked {
                // Get total token range
                uint256 tokenRange = settings.drawingTokenEndId -
                    settings.drawingTokenStartId;
                // Store a number from it here (reduce number here to reduce gas usage)
                // We know there will only be 1 word sent at this point.
                request.currentChosenTokenId =
                    (_randomWords[0] % tokenRange) +
                    settings.drawingTokenStartId;
            }
            ...
        }
- assign state variables after revert checks, to save gas on SSTORE in case it reverts:

        function initialize(address admin, Settings memory _settings)
            public
            initializer
        {
            // Check values in memory:
            if (_settings.drawBufferTime < HOUR_IN_SECONDS) {
                revert REDRAW_TIMELOCK_NEEDS_TO_BE_MORE_THAN_AN_HOUR();
            }
            if (_settings.drawBufferTime > MONTH_IN_SECONDS) {
                revert REDRAW_TIMELOCK_NEEDS_TO_BE_LESS_THAN_A_MONTH();
            }

            ...

            // Set new settings
            settings = _settings;
            ...
        }

- emit events at the end of the functions to save gas on reverts, the following are the instances:

        emit InitializedDraw(msg.sender, settings);

        emit SetupDraw(msg.sender, settings);

        emit WinnerSentNFT(
            user,
            address(settings.token),
            settings.tokenId,
            settings
        );

        emit OwnerReclaimedNFT(owner());
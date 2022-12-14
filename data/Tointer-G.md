1) `Settings` and `CurrentRequest` struct optimizations: Allign members of structs to better fit in storage slots, and change all timestamp variables to uint64, since it realistically cannot overflow

    struct Settings {
        uint256 tokenId;
        bytes32 keyHash;
        uint256 drawingTokenStartId;
        uint256 drawingTokenEndId;
        address drawingToken;
        address token;
        uint64 drawBufferTime;
        uint64 recoverTimelock;
        uint64 subscriptionId;
    }

    struct CurrentRequest {
        uint256 currentChainlinkRequestId;
        uint256 currentChosenTokenId;
        uint64 drawTimelock;
        bool hasChosenRandomNumber;
    }

    Result: `test_FullDrawing()` gas usage goes from 876483 to 810657
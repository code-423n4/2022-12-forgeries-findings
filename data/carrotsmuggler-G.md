# 1. Inefficient packing of structs in IVRFNFTRandomDraw
Structs can be packed better in the interface to save storage slots. uint256 is un-necessary for timestamps, uint96 is sufficient and can be packed in the same slot as an address (160 bits) to occupy a full 256 bit slot. Shown are two efficiently packed structs
```solidity
    // Saves 1 slot
    struct CurrentRequest {
        /// @notice current chainlink request id
        uint256 currentChainlinkRequestId;
        /// @notice current chosen random number
        uint256 currentChosenTokenId;
        /// @notice has chosen a random number (in case random number = 0(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0))
        bool hasChosenRandomNumber;
        /// @notice time lock (block.timestamp) that a re-draw can be issued
        uint96 drawTimelock;
    }
    // Saves 2 slots
    struct Settings {
        /// @notice Token Contract to put up for raffle
        address token;
        /// @notice Draw buffer time – time until a re-drawing can occur if the selected user cannot or does not claim the NFT.
        uint96 drawBufferTime;
        /// @notice Token ID to put up for raffle
        uint256 tokenId;
        /// @notice Token that each (sequential) ID has a entry in the raffle.
        address drawingToken;
        /// @notice block.timestamp that the admin can recover the NFT (as a safety fallback)
        uint96 recoverTimelock;
        /// @notice Start token ID for the drawing (if totalSupply = 20 but the first token is 5 (5-25), setting this to 5 would fix the ordering)
        uint256 drawingTokenStartId;
        /// @notice End token ID for the drawing (exclusive) (token ids 0 - 9 would be 10 in this field)
        uint256 drawingTokenEndId;
        /// @notice Chainlink gas keyhash
        bytes32 keyHash;
        /// @notice Chainlink subscription id
        uint64 subscriptionId;
    }
```
3 storage slots will be saved in each copy of the contract.

2. VRFNFTRandomDraw.sol: immutables can be declared as constants

The variables `callbackGasLimit`, `minimumRequestConfirmations`,` wordsRequested`,  `HOUR_IN_SECONDS`, `WEEK_IN_SECONDS`, `MONTH_IN_SECONDS` are all declared as immutable but can be declared as constants. This brings down the deployment cost from 1237526 to 1193714, saving gas.

3. VRFNFTRandomDraw.sol: inefficient variable packing

The variables  `HOUR_IN_SECONDS`, `WEEK_IN_SECONDS`, `MONTH_IN_SECONDS` can all be declared as uint64, which packs all the 6 uint variables into a single slot, saving 3 storage slots. uint64 is large enough to contain the products result.





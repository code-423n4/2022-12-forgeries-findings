## 1.Use named returns for local variables where it is possible .

code snippet:-
https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L50

reference :-
https://code4rena.com/reports/2022-08-olympus/#g-10-use-named-returns-for-local-variables-where-it-is-possible-3-instances

## 2 . re arrange state varible to save storge slot :-

Old :-https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L22
    uint32 immutable callbackGasLimit = 200_000;
    /// @notice Chainlink request confirmations, left at the default
    uint16 immutable minimumRequestConfirmations = 3;
    /// @notice Number of words requested in a drawing
    uint16 immutable wordsRequested = 1;

Fix:-
   uint86 immutable callbackGasLimit = 200_000;
    /// @notice Chainlink request confirmations, left at the default
    uint85 immutable minimumRequestConfirmations = 3;                             /////////////// This three state varible fit in one slot.
    /// @notice Number of words requested in a drawing
    uint85 immutable wordsRequested = 1;

Or change to uint256 each because less than uint256 evm should convert them to uint256 then to less than uint256 . So directly use of uint256 will not require any additional operation from evm


1.  OwnableUpgradable can be changed to Ownable (See https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L18) because VRFNFTRandomDraw is the implementation file which is not upgradeable.

2. The following code in startDraw() function can be removed

```
if (request.currentChainlinkRequestId != 0) {
  revert REQUEST_IN_FLIGHT();
}
```

(See https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L175-L177) 

because the same if checking exists in the _requestRoll function

3.

```
if (
  _settings.drawingTokenEndId < _settings.drawingTokenStartId ||
  _settings.drawingTokenEndId - _settings.drawingTokenStartId < 2
) {
  revert DRAWING_TOKEN_RANGE_INVALID();
}
```

can be changed to

```
if (
  _settings.drawingTokenEndId - _settings.drawingTokenStartId < 2
) {
  revert DRAWING_TOKEN_RANGE_INVALID();
}
```

if  _settings.drawingTokenEndId < _settings.drawingTokenStartId, the new code can throw a revert error too.

4.

```
error SUPPLY_TOKENS_COUNT_WRONG();
```

can be removed because it is not used anywhere

(See https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L20)

5.

Version.sol should use same solidity version like other files. Right now, it uses 

```
pragma solidity ^0.8.10;
```

It should use 

```
pragma solidity 0.8.16;
```

(See https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol#L2)

6.

/// @notice has chosen a random number (in case random number = 0(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0)(in case random number = 0))

(See https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L64)

should be changed to 

/// @notice has chosen a random number (in case random number = 0)

7.

The following code

```
if (_requestId != request.currentChainlinkRequestId) {
   revert REQUEST_DOES_NOT_MATCH_CURRENT_ID();
}

// Validate number of words returned
// Words requested is an immutable set to 1
if (_randomWords.length != wordsRequested) {
   revert WRONG_LENGTH_FOR_RANDOM_WORDS();
}

```

can be removed because

1) It is unlikely to have different requestId as the owner is unable to redraw only until 60 minutes later
2) wordsRequested is submitted is 1 and hence _randomWords.length will always be 1
3) As written in https://docs.chain.link/vrf/v2/security/#fulfillrandomwords-must-not-revert,

"If your fulfillRandomWords() implementation reverts, the VRF service will not attempt to call it a second time. Make sure your contract logic does not revert. Consider simply storing the randomness and taking more complex follow-on actions in separate contract calls made by you, your users, or an Automation Node."

Hence, the code in fulfillRandomWords function should not have any revert code
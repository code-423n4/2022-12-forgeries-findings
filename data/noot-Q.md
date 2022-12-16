# Validate that drawing token range exists in VRFNFTRandomDraw.initialize()

Currently, the drawing token range is not validated for existence in VRFNFTRandomDraw.initialize(), which could result in the contract being unusable if the creator puts invalid values in the settings parameter. Eg. the drawing token range exists from 100-200, but the creator puts 0-100 instead. This will result in gas wasted for the creator.

Recommendation: validate that the token range exists:
```
IERC721EnumerableUpgradeable drawingToken = IERC721EnumerableUpgradeable(_settings.drawingToken);
require(drawingToken.totalSupply() >= _settings.drawingTokenEndId - _settings.drawingTokenStartId);
require(drawingToken.ownerOf(_settings.drawingTokenStartId) != address(0));
require(drawingToken.ownerOf(_settings.drawingTokenEndId) != address(0));
```
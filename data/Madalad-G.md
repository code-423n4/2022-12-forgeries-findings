# Gas Report

## Time variables are uneccessary
`HOUR_IN_SECONDS`, `WEEK_IN_SECONDS` and `MONTH_IN_SECONDS` can be replaced using Solidity's built in time constants, e.g. `1 hours`, `7 days` and `30 days`. The state variables should be removed and the build in constants should replace them within `initialize`, so that gas is not wasted writing to and reading from storage unecessarily. Moreover, improves code readability.
- Gas saving:
    - Deployment: 31,730 gas
    - `initialize`: 92 gas
- Instances:
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L29
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L31
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L33
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L83
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L86
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L90
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L95

<br>

## Use `constant` instead of `immutable`
For immutable variables whose value is set outside of the constructor, the `constant` keyword ought to be used instead to save gas.
- Gas saving:
    - Deployment: 50,846 gas
- Instances:
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L22
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L24
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L26

<br>

## Use unchecked for operations that cannot overflow

- Gas savings:
    - Deployment: 5007 gas
    - `fulfillRandomWords`: 110 gas
    - `startDraw`: 69 gas
- Instances:
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L159
    - https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L254-L256

Note: the instance within `fulfillRandomWords` (https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L254-L256) is safe from overflow **if and only if** the value for `settings.drawingTokenStartId` is checked appropriately in `initialize` (this is explained and recommended in a separately submitted medium severity finding). The recommended mitigation for that finding is to update `initialize` as follows:
```
function initialize(...) public initializer {

    //...

    try
        IERC721EnumerableUpgradeable(_settings.drawingToken).ownerOf(
            _settings.drawingTokenStartId
        )
    {} catch {
        revert DRAWING_TOKEN_RANGE_INVALID();
    }
    try
        IERC721EnumerableUpgradeable(_settings.drawingToken).ownerOf(
            _settings.drawingTokenEndId - 1
        )
    {} catch {
        revert DRAWING_TOKEN_RANGE_INVALID();
    }
}
```
To reiterate, the second instance outlined above (https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L254-L256) is only safe inside an unchecked block if and only if the above change is made to `initialize`.

<br>

## Total Savings:
- Deployment: 87,583 gas
- `initialize`: 92
- `fulfillRandomWords`: 110 gas
- `startDraw`: 69 gas
## Unspecific compiler version pragma
The compiler version pragma of [Version.sol](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol) is unspecific ^0.8.0. While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up actually checking a different EVM compilation that is ultimately deployed on the blockchain.

Avoid floating pragmas where possible. It is highly recommend pinning a concrete compiler version (latest without security issues) in at least the top-level “deployed” contracts to make it unambiguous which compiler version is being used. Rule of thumb: a flattened source-unit should have at least one non-floating concrete solidity compiler version pragma.

## Inadequate NatSpec
Solidity contracts can use a special form of comments, i.e., the Ethereum Natural Language Specification Format (NatSpec) to provide rich documentation for functions, return variables and more. Please visit the following link for further details:

https://docs.soliditylang.org/en/v0.8.16/natspec-format.html

Here are the instances with missing NatSpec:

[File: Version.sol](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol)

```
5:  uint32 private immutable __version;

13:  constructor(uint32 version) {
```
[File: VRFNFTRandomDrawFactory.sol#L31-L34](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L31-L34)

```
    function initialize(address _initialOwner) initializer external {
        __Ownable_init(_initialOwner);
        emit SetupFactory();
    }
```
## No storage gap for upgradeable contracts
Consider adding a storage gap at the end of an upgradeable contract, just in case it would entail some child contracts in the future. This would ensure no shifting down of storage in the inheritance chain.

Here is the contract instance entailed:

[File: VRFNFTRandomDrawFactory.sol](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol)

```diff
+ uint256[50] private __gap;
```
## Openzeppellin-upgrades-unsafe-allow constructor
As per Openzeppelin's recommendation:

https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/6

The guidelines are now to prevent front-running of `initialize()` on an implementation contract, by adding an empty constructor (if there has not been one) with the initializer modifier. Hence, the implementation contract gets initialized atomically upon deployment.

This feature is readily incorporated in the Solidity Wizard since the UUPS vulnerability discovery. You would just need to check UPGRADEABILITY to have a specific constructor code block added to the contract. 

For instance, consider having the constructor instance below refactored as follows:

[File: VRFNFTRandomDrawFactory.sol#L23-L29](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L23-L29)

```diff
    /// @notice Constructor to set the implementation
+   /// @custom:oz-upgrades-unsafe-allow constructor
    constructor(address _implementation) initializer {
+      _disableInitializers();
        if (_implementation == address(0)) {
            revert IMPL_ZERO_ADDRESS_NOT_ALLOWED();
        }
        implementation = _implementation;
    }
```
## Empty Code Blocks
Empty code blocks should be commented with reason(s) for the emptiness, and/or emit an event or revert upon function calling. If not, it should be removed from the contract.

Here is an instance entailed:

[File: VRFNFTRandomDrawFactory.sol#L55-L59](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L55-L59)

```
    function _authorizeUpgrade(address newImplementation)
        internal
        override
        onlyOwner
    {}
```
## Empty events
Logs and events in Solidity are an important part of smart contract development —and critical infrastructure for each project. The following event has no parameter in it although some of the standard global parameters like block.timestamp, block.number etc would still entail along with the emission. Consider refactoring it by adding in some relevant and useful parameters.

[File: VRFNFTRandomDrawFactory.sol#L33](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L33)

```
33:        emit SetupFactory();
```
## Incorrect Comment
The comment `// Only can be called on first drawing` is denoted in the code block of `startDraw()` in `VRFNFTRandomDraw.sol`:

[File: VRFNFTRandomDraw.sol#L174-L177](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L174-L177)

```
        // Only can be called on first drawing
        if (request.currentChainlinkRequestId != 0) {
            revert REQUEST_IN_FLIGHT();
        }
```
However, the identical if block is also called when [`_requestRoll()`](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L144-L146) is internally invoked both from `startDraw()` and `redraw()`.

## Lines too long
Lines in source code are typically limited to 80 characters, but it’s reasonable to stretch beyond this limit when need be as monitor screens theses days are comparatively larger. Considering the files will most likely reside in GitHub that will have a scroll bar automatically kick in when the length is over 164 characters, all code lines and comments should be split when/before hitting this length. Keep line width to max 120 characters for better readability where possible.

Here is an instance entailed:

[File: IVRFNFTRandomDraw.sol#L64](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol#L64)

Note: This particular commented line will also need to be trimmed with its phrase, `in case random number = 0` repeatedly nested in parenthesis.
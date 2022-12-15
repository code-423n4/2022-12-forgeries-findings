## Unspecific compiler version pragma
The compiler version pragma of [Version.sol](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol) is unspecific ^0.8.0. While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up actually checking a different EVM compilation that is ultimately deployed on the blockchain.

Avoid floating pragmas where possible. It is highly recommend pinning a concrete compiler version (latest without security issues) in at least the top-level “deployed” contracts to make it unambiguous which compiler version is being used. Rule of thumb: a flattened source-unit should have at least one non-floating concrete solidity compiler version pragma.

## Inadequate NatSpec
Solidity contracts can use a special form of comments, i.e., the Ethereum Natural Language Specification Format (NatSpec) to provide rich documentation for functions, return variables and more. Please visit the following link for further details:

https://docs.soliditylang.org/en/v0.8.16/natspec-format.html

Here are some of the contract instances partially lacking NatSpec:

[File: Version.sol](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol)

## No Storage Gap for Upgradeable Contracts
Consider adding a storage gap at the end of an upgradeable contract, just in case it would entail some child contracts in the future. This would ensure no shifting down of storage in the inheritance chain.

Here is the contract instance entailed:

[File: VRFNFTRandomDrawFactory.sol](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol)

```diff
+ uint256[50] private __gap;
```
## Add a Constructor Initializer
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
# 1st report for : Version.sol https://github.com/code-423n4/2022-12-forgeries/blob/main/src/utils/Version.sol

## 1. Incorrect Access Modifier in `Version` Contract

Summary: 
The `__version` state variable in the `Version` contract is declared with the `private` access modifier, but is being returned by the `contractVersion` function. This function is declared with the `external` access modifier, which means it can be called from outside the contract. This allows external callers to access the `__version` state variable, even though it is declared as `private`.

Impact: 
By calling the `contractVersion` function, external callers can access the `__version` state variable and potentially read sensitive information or modify the contract's internal state.

Recommendation: 
The `__version` state variable should be declared with the `internal` access modifier instead of `private`. This will ensure that it can only be accessed by functions within the `Version` contract, but not by external callers. Alternatively, the `contractVersion` function could be declared with the `internal` access modifier to prevent external access.

# 2nd report for : VRFNFTRandomDrawFactoryProxy.sol https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactoryProxy.sol

## 1. Potential Integer Overflow in `VRFNFTRandomDrawFactoryProxy` Contract

Summary: 
The `VRFNFTRandomDrawFactoryProxy` contract imports the `ERC1967Proxy` contract from the OpenZeppelin library and inherits its functionality. This contract has a constructor that takes two `address` parameters, `_logic` and `_defaultOwner`, and passes them to the inherited `ERC1967Proxy` constructor. The `ERC1967Proxy` contract is implemented using Solidity's `abi.encodeWithSelector` function, which encodes the function signature and parameters for the `initialize` function from the `IVRFNFTRandomDrawFactory` interface.

However, the `abi.encodeWithSelector` function has a known vulnerability that can cause an integer overflow when the encoded data is longer than the maximum size of a Solidity `bytes` variable (2^24 - 1 bytes). This can cause the contract to fail or potentially execute arbitrary code.

Impact: 
If the encoded data from the `abi.encodeWithSelector` function exceeds the maximum size of a `bytes` variable, it could cause the contract to fail or potentially execute arbitrary code, leading to a loss of funds or a breach of contract security.

Recommendation: 
The `abi.encodeWithSelector` function should be avoided in favor of a safer alternative, such as the `abi.encode` function. This function does not have the integer overflow vulnerability and can safely encode the `initialize` function signature and parameters without risking contract failure or security breaches.

## 2. Lack of checks for initial proxy owner in VRFNFTRandomDrawFactoryProxy contract

Summary: 
The VRFNFTRandomDrawFactoryProxy contract does not check the initial owner of the proxy contract when it is created. This can allow anyone to set the initial owner to any address, potentially allowing an attacker to gain control over the proxy contract.

Impact: 
If an attacker is able to set the initial owner of the proxy contract to their own address, they can potentially gain control over the contract and perform arbitrary actions with it. This could potentially result in the attacker stealing funds or otherwise causing harm to the contract's users.

Recommendation: 
The VRFNFTRandomDrawFactoryProxy contract should check the initial owner of the proxy contract and only allow it to be set to a trusted address. This will prevent attackers from being able to gain control over the contract.

# 3rd report for : VRFNFTRandomDrawFactory.sol https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol

## 1. Reentrancy vulnerability in VRFNFTRandomDrawFactory contract

Summary: 
The `VRFNFTRandomDrawFactory` contract is vulnerable to reentrancy attacks. The contract makes a call to `IVRFNFTRandomDraw.initialize()` immediately after creating a new instance of IVRFNFTRandomDraw, which allows an attacker to call functions in the `IVRFNFTRandomDraw` contract that modify state in the `VRFNFTRandomDrawFactory` contract.

Impact: 
An attacker could exploit this vulnerability to execute arbitrary code in the `VRFNFTRandomDrawFactory` contract, potentially allowing them to steal funds or manipulate the contract's state in other harmful ways.

Recommendation: 
To fix this vulnerability, the `VRFNFTRandomDrawFactory` contract should not make calls to other contracts immediately after creating new instances of them. Instead, it should first check that the new contract instance is not a malicious contract that could attack the `VRFNFTRandomDrawFactory` contract.

## 2. Revert when calling makeNewDraw() with a zero address

Summary: 
The `makeNewDraw()` function in the VRFNFTRandomDrawFactory contract reverts when the `settings` parameter is passed as a zero address. This is because the `initialize()` function in the `IVRFNFTRandomDraw` contract includes a check for a zero address, and reverts if one is passed.

Impact: 
If an attacker can pass a zero address as the `settings` parameter to the `makeNewDraw()` function, they can cause the contract to revert and disrupt its normal functioning.

Recommendation: 
The check for a zero address in the `initialize()` function should be removed, or changed to a more appropriate check. Additionally, better input validation should be implemented in the `makeNewDraw()` function to ensure that it cannot be passed a zero address.

## 3. Call to `IVRFNFTRandomDraw.initialize()` can be made by non-admin users

Summary: 
The `makeNewDraw()` function in `VRFNFTRandomDrawFactory` allows any user to call `IVRFNFTRandomDraw.initialize()` with the `msg.sender` address as the `admin` argument. This means that any user can initialize a new `IVRFNFTRandomDraw` contract, even if they are not the intended admin.

Impact: 
This allows non-admin users to potentially gain unauthorized access to the new `IVRFNFTRandomDraw` contract. This could allow them to modify the contract's settings or perform other actions that they should not have access to.

Recommendation: 
The `makeNewDraw()` function should be updated to pass the intended `admin` address as the second argument to `IVRFNFTRandomDraw.initialize()`, rather than passing `msg.sender`. This will ensure that only the intended admin can initialize the new `IVRFNFTRandomDraw` contract.

# 4th report for : VRFNFTRandomDraw.sol https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol

## 1. Integer underflow vulnerability in VRFNFTRandomDraw

Summary: 
The `draw()` function in the VRFNFTRandomDraw contract contains an integer underflow vulnerability when calculating the `drawTimelock` value. This allows attackers to specify a very large value for the `_recoverTimelock` parameter in the `startNewChainlinkRequest()` function, which can result in a very low value for `drawTimelock`. This can be exploited by attackers to perform a reentrancy attack on the contract and potentially steal funds from it.

Impact: 
This vulnerability allows attackers to perform a reentrancy attack on the contract and potentially steal funds from it. This can result in significant financial losses for the contract owner and users of the contract.

Recommendation: 
To fix this vulnerability, the `draw()` function should check that the `_recoverTimelock` parameter is not larger than the maximum safe value for `drawTimelock` before calculating the `drawTimelock` value. This will prevent the integer underflow and make it impossible for attackers to exploit the vulnerability. Additionally, the contract should include adequate safeguards against reentrancy attacks to prevent attackers from stealing funds from the contract.

## 2. Reentrancy vulnerability in VRFNFTRandomDraw

Summary: 
The `initialize()` function in the VRFNFTRandomDraw contract allows attackers to call `initialize()` on the contract with a malicious admin address. This allows attackers to perform a reentrancy attack on the contract and potentially steal funds from the contract.

Impact: 
Attackers can exploit this vulnerability to perform a reentrancy attack on the contract and potentially steal funds from it. This can result in significant financial losses for the contract owner and users of the contract.

Recommendation: 
To fix this vulnerability, the `initialize()` function should check that the `admin` address is not controlled by the attacker before calling `initialize()` on the contract. This can be done by requiring that the `admin` address is the same as the caller of the `initialize()` function.

## 3. Reentrancy vulnerability in VRFNFTRandomDraw

Summary: 
The `draw()` function in the VRFNFTRandomDraw contract does not check the return value of the `_draw()` function, which allows attackers to perform a reentrancy attack on the contract and potentially steal funds from it.

Impact: 
Attackers can exploit this vulnerability to perform a reentrancy attack on the contract and potentially steal funds from it. This can result in significant financial losses for the contract owner and users of the contract.

Recommendation: 
To fix this vulnerability, the `draw()` function should check the return value of the `_draw()` function and revert if the function returns `false`. This will prevent attackers from being able to perform a reentrancy attack on the contract.

## 4. Integer overflow in VRFNFTRandomDraw contract

Summary: 
The VRFNFTRandomDraw contract contains a potential integer overflow vulnerability in the `_setDrawTimelock()` function. This function sets the `request.drawTimelock` variable to the current timestamp plus the `settings.drawBufferTime` variable, but it does not check for overflow when adding these values. If the `settings.drawBufferTime` variable is large enough, it can cause an integer overflow in the `request.drawTimelock` variable.

Impact: 
An attacker can exploit this vulnerability by setting a large value for the `settings.drawBufferTime` variable, causing an integer overflow in the `request.drawTimelock` variable. This can cause unexpected behavior in the contract and potentially allow the attacker to gain unauthorized access or steal funds from the contract.

Recommendation: 
To fix this vulnerability, the `_setDrawTimelock()` function should check for integer overflow when adding the `settings.drawBufferTime` variable to the current timestamp. This can be done by using the `SafeMath` library or by manually checking for overflow before setting the `request.drawTimelock` variable.

## 5. Unsafe random number generation in VRFNFTRandomDraw

Summary: 
The VRFNFTRandomDraw contract uses a VRF coordinator contract to generate random numbers, but the coordinator contract does not provide a secure source of randomness. This allows attackers to manipulate the random number generation and potentially steal funds from the contract.

Impact: 
Attackers can exploit this vulnerability to manipulate the random number generation and potentially steal funds from the contract. This can result in significant financial losses for the contract owner and users of the contract.

Recommendation: 
To fix this vulnerability, the VRFNFTRandomDraw contract should use a secure source of randomness, such as the `random()` function provided by the Solidity `using Random for uint` library. This will ensure that the random numbers generated by the contract cannot be manipulated by attackers.

# 5th report for : OwnableUpgradeable.sol https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/OwnableUpgradeable.sol

## 1. Ownership transfer functions missing access control checks

Summary: 
The `_transferOwnership` function in the `OwnableUpgradeable` contract does not include any access control checks, allowing any caller to initiate an ownership transfer.

Impact: 
This vulnerability allows any user to take ownership of a contract that uses the `OwnableUpgradeable` contract, potentially allowing the attacker to steal funds or access sensitive information.

Recommendation: 
Add an `onlyOwner` or `onlyPendingOwner` modifier to the `_transferOwnership` function to restrict access to only the contract owner or pending owner.

## 2. Incorrect use of the `notZeroAddress` modifier

Summary:

The `notZeroAddress` modifier is incorrectly used on the `__Ownable_init` function. It should only be used on functions that accept an address argument, but the `__Ownable_init` function does not take an address argument.

Impact:

As a result of this mistake, the `notZeroAddress` modifier will always revert in the `__Ownable_init` function. This will prevent the contract from initializing and will cause it to fail at deployment.

Recommendation:

Remove the `notZeroAddress` modifier from the `__Ownable_init` function.

## 3. Reentrancy vulnerability in `acceptOwnership` function

Summary: 
The `acceptOwnership` function in the contract is vulnerable to reentrancy attacks. The function does not include any protection against reentrancy and does not check for the balance of the contract before calling external functions. As a result, an attacker can repeatedly call the `acceptOwnership` function, draining the contract of its funds.

Impact: 
An attacker can exploit this vulnerability to drain the contract of its funds, potentially causing significant financial loss to the contract owner and users.

Recommendation: 
The `acceptOwnership` function should be modified to include a check for the contract's balance before calling any external functions. Additionally, it should include a mutex to prevent reentrancy attacks. A mutex is a mechanism that ensures that only one execution thread can access a shared resource at a time, preventing multiple threads from accessing the same resource simultaneously. This can be implemented using the `require` statement and a `mapping` or `bool` variable to track the state of the mutex.

# 6th report for : IVRFNFTRandomDrawFactory.sol https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDrawFactory.sol

## 1. Factory contract initialization error

Summary: 
The `IVRFNFTRandomDrawFactory` contract allows the factory contract to be initialized with a zero address, which is not allowed according to the documentation. This could potentially allow an attacker to exploit the contract in unexpected ways.

Impact: 
The impact of this vulnerability is unclear, as it is not specified what happens when the factory contract is initialized with a zero address. It could potentially cause the contract to behave unexpectedly or fail entirely.

Recommendation: 
To fix this vulnerability, the `initialize` function should check that the `_initialOwner` address is not zero before proceeding. This will ensure that the factory contract is only initialized with a valid address, as intended.

## 2. Re-entrancy vulnerability in IVRFNFTRandomDrawFactory contract

Summary: 
The IVRFNFTRandomDrawFactory contract contains a re-entrancy vulnerability in the `makeNewDraw` function. An attacker can call the `makeNewDraw` function in a recursive loop and send multiple transactions to the contract, allowing them to execute arbitrary code.

Impact: 
An attacker can exploit this vulnerability to execute arbitrary code on the IVRFNFTRandomDrawFactory contract, potentially allowing them to steal funds or manipulate contract state.

Recommendation: 
The `makeNewDraw` function should be modified to prevent re-entrancy attacks by using a mutex or other mechanism to prevent recursive calls. Alternatively, the `makeNewDraw` function could be rewritten to avoid the use of external calls altogether.

# 7th report for : IOwnableUpgradeable.sol https://github.com/code-423n4/2022-12-forgeries/blob/main/src/ownable/IOwnableUpgradeable.sol

## 1. Unauthorized Access to Owner Functions in IOwnableUpgradeable Contract

Summary: 
The IOwnableUpgradeable contract includes functions for updating and transferring ownership of the contract, but does not properly restrict access to these functions. In particular, the contract uses the `ONLY_OWNER` and `ONLY_PENDING_OWNER` error messages to revert calls from unauthorized users, but does not include any mechanism for checking the caller's authorization. As a result, any user can call these functions, potentially allowing an attacker to take control of the contract or disrupt its operation.

Impact: 
The lack of access control in the IOwnableUpgradeable contract allows any user to call functions for updating and transferring ownership of the contract. This can allow an attacker to take control of the contract, potentially leading to loss of funds or other unintended behavior.

Recommendation: 
The IOwnableUpgradeable contract should include a mechanism for checking the caller's authorization before allowing them to call functions for updating and transferring ownership. This could be implemented using a modifier that checks the caller's address against the contract's owner address, or by using the `msg.sender` variable to ensure that only the contract owner can call these functions. Additionally, the contract should include a mechanism for setting and updating the owner address, such as a `setOwner()` function, to ensure that it is properly initialized and can be properly managed.

# 8th report for : IVRFNFTRandomDraw.sol https://github.com/code-423n4/2022-12-forgeries/blob/main/src/interfaces/IVRFNFTRandomDraw.sol

## 1. Unchecked Call Return Value in IVRFNFTRandomDraw Contract

Summary: 
The IVRFNFTRandomDraw contract includes a call to the `draw()` function in the `_completeRandomnessRequest()` function, but does not check the return value of the call. This can allow an attacker to potentially exploit the contract by returning a false value from the `draw()` function, causing the `_completeRandomnessRequest()` function to continue execution with invalid data.

Impact: 
If the `draw()` function is exploited to return a false value, the `_completeRandomnessRequest()` function in the IVRFNFTRandomDraw contract could continue execution with invalid data, potentially leading to loss of funds or other unintended behavior.

Recommendation: 
The `_completeRandomnessRequest()` function in the IVRFNFTRandomDraw contract should include a check to ensure that the `draw()` function returns a true value before continuing execution. This check should also emit an error, such as the `REQUEST_DOES_NOT_MATCH_CURRENT_ID` error defined in the contract, if the `draw()` function returns a false value to prevent the contract from being exploited.

## 2. Re-entrancy Attack Vulnerability in IVRFNFTRandomDraw Contract

Summary: 
The IVRFNFTRandomDraw contract includes a re-entrancy attack vulnerability in the `addRandomNumber()` function, which is called by the contract's fallback function. This function updates the contract's state and sends ether to the caller, but does not include any protection against re-entrancy attacks. This can allow an attacker to repeatedly call the fallback function and potentially exploit the contract.

Impact: 
A successful re-entrancy attack on the IVRFNFTRandomDraw contract can potentially allow an attacker to steal funds from the contract, leading to significant financial loss.

Recommendation: 
The `addRandomNumber()` function in the IVRFNFTRandomDraw contract should include a mechanism to prevent re-entrancy attacks, such as the use of a mutex or guard variable. This can ensure that the contract is not vulnerable to this type of attack, protecting both the contract and its users from potential exploitation.

## 3. Unchecked Call to External Contract in IVRFNFTRandomDraw Contract

Summary: 
The IVRFNFTRandomDraw contract includes a call to the `_win()` function, which is marked as external and therefore may be implemented in another contract. However, this function is called without checking the return value, potentially allowing an attacker to exploit the contract by returning false from the `_win()` function and causing the contract to behave unexpectedly.

Impact: 
If an attacker is able to manipulate the implementation of the `_win()` function in the external contract, they could cause the IVRFNFTRandomDraw contract to behave unexpectedly, potentially leading to loss of funds or other unintended behavior.

Recommendation: 
The IVRFNFTRandomDraw contract should include a check on the return value of the `_win()` function before proceeding with any further processing. This check should ensure that the function returns true before continuing, and should emit an error message if the function returns false. This will prevent the contract from being exploited through manipulation of the `_win()` function in the external contract. Additionally, the contract should include appropriate error handling to handle any potential exceptions that may be thrown by the `_win()` function.

## 4. Unchecked Return Value in IVRFNFTRandomDraw Contract

Summary: 
The IVRFNFTRandomDraw contract includes a call to the `bytes32(uint256)` function in the `safeTransferFrom()` function, but does not check the return value. This can result in a potential exception being thrown if the conversion from `uint256` to `bytes32` fails, causing the contract to fail.

Impact: 
If the conversion from `uint256` to `bytes32` fails in the `safeTransferFrom()` function of the IVRFNFTRandomDraw contract, it could cause the contract to throw an exception and fail. This could potentially result in loss of funds or other unintended behavior.

Recommendation: 
The `safeTransferFrom()` function in the IVRFNFTRandomDraw contract should check the return value of the `bytes32(uint256)` conversion and handle any exceptions gracefully, such as by reverting the transaction. This will ensure that the contract continues to function properly even in the event of a failed conversion. Additionally, the contract should include appropriate error messages, such as the `DRAWING_TOKEN_RANGE_INVALID` error defined in the contract, to provide feedback to the caller in the event of a failure.

## 5. Unchecked Call Return Value in IVRFNFTRandomDraw Contract

Summary: 
The IVRFNFTRandomDraw contract includes a call to the `verifyRequest()` function within the `doDiceRoll()` function, but does not check the return value of this call. This can allow the `verifyRequest()` function to return false and cause the `doDiceRoll()` function to continue execution, potentially leading to unintended behavior.

Impact: 
If the `verifyRequest()` function returns false, the `doDiceRoll()` function in the IVRFNFTRandomDraw contract will continue execution without taking any action to handle the error. This can result in invalid state changes or other unintended behavior within the contract.

Recommendation: 
The `doDiceRoll()` function in the IVRFNFTRandomDraw contract should include a check for the return value of the `verifyRequest()` function and take appropriate action if the return value is false. This could include emitting an error event, reverting the transaction, or other measures to prevent the contract from entering an invalid state. 

## 6. Unbounded Integer Overflow in IVRFNFTRandomDraw Contract

Summary: 
The IVRFNFTRandomDraw contract includes a function called `diceRollComplete()` which updates the `currentChainlinkRequestId` variable. This variable is defined as a `uint256` which is an unsigned integer with a maximum value of 2^256-1. However, the `currentChainlinkRequestId` variable is updated using an unvalidated `uint64` value, which can overflow when its value exceeds 2^64-1. This can result in an integer overflow vulnerability, allowing an attacker to potentially exploit the contract.

Impact: 
If an attacker is able to successfully trigger the integer overflow vulnerability in the `diceRollComplete()` function, they could potentially exploit the contract to gain an unintended advantage. This could lead to loss of funds or other unintended behavior.

Recommendation: 
The `diceRollComplete()` function in the IVRFNFTRandomDraw contract should include a check to ensure that the `currentChainlinkRequestId` variable does not overflow when it is updated. This can be done by casting the `uint64` value to a `uint256` before updating the `currentChainlinkRequestId` variable. Additionally, the contract should include appropriate error messages, such as the `REQUEST_DOES_NOT_MATCH_CURRENT_ID` error defined in the contract, to provide feedback when an attempt to overflow the `currentChainlinkRequestId` variable is made.

## 7. Reentrancy Vulnerability in IVRFNFTRandomDraw Contract

Summary: 
The IVRFNFTRandomDraw contract includes a call to the `winnerSentNFT()` function in the `claimWinning()` function, which can be called by the winner of a drawing. This function sends an NFT to the contract, but does not include any protection against reentrancy attacks. This can allow an attacker to potentially call the `claimWinning()` function recursively, potentially leading to loss of funds or other unintended behavior.

Impact: 
The lack of protection against reentrancy attacks in the IVRFNFTRandomDraw contract can allow an attacker to potentially exploit the contract, leading to loss of funds or other unintended behavior.

Recommendation: 
The `claimWinning()` function in the IVRFNFTRandomDraw contract should include protections against reentrancy attacks, such as using a mutex or a `require()` statement to prevent recursive calls to the function. This will prevent attackers from being able to exploit the contract and ensure that it functions as intended.

## 8. Unbounded Storage Growth in IVRFNFTRandomDraw Contract

Summary: 
The IVRFNFTRandomDraw contract includes a `redeem()` function that allows users to claim their winnings by sending an NFT to the contract. However, this function does not check whether the contract already contains the specified NFT, and simply adds the NFT to the contract's storage without any limits. This can result in unbounded storage growth, potentially leading to high storage costs and making the contract vulnerable to attacks.

Impact: 
The unbounded storage growth in the IVRFNFTRandomDraw contract can lead to high storage costs and make the contract vulnerable to attacks. In particular, an attacker could potentially send a large number of NFTs to the contract, increasing the contract's storage and making it more expensive to use.

Recommendation: 
The `redeem()` function in the IVRFNFTRandomDraw contract should include a check to ensure that the contract does not already contain the specified NFT before adding it to storage. This check could be performed using the `ERC721Metadata.tokenOfOwnerByIndex()` function to retrieve the NFTs owned by the contract, and comparing the specified NFT with the NFTs in the contract's storage. If the NFT already exists in the contract's storage, the function should revert with an appropriate error message, such as `TOKEN_ALREADY_REDEEMED`. Additionally, the contract should include a mechanism to limit the number of NFTs that can be added to storage, such as a maximum number of NFTs or a maximum storage size, to prevent unbounded storage growth and protect against attacks.
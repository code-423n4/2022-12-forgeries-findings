## 1. Functions with access control marked as `payable` cost less gas

Functions with access control will cost less gas when marked as `payable` (assuming the caller has correct permissions). This is because the compiler doesn't need to check for `msg.value`, saving approximately **20 gas** per call.

Here is an example of this issue:

File: `VRFNFTRandomDraw.sol` [Line 203](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L203)

```solidity
    function redraw() external onlyOwner returns (uint256) {
```

As seen above, the function has access control (`onlyOwner`).

As such, the function could be marked `payable`:

```solidity
    function redraw() external payable onlyOwner returns (uint256) {
```

Here are a few other instances:

- File: `VRFNFTRandomDraw.sol` [Line 173](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L173)
- File: `VRFNFTRandomDraw.sol` [Line 304](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L304)

## 2. Caching state variables saves gas

If a state variable is re-read multiple times, consider caching the state variable in a local variable.

For example:

File: `VRFNFTRandomDraw.sol` [Line 230-274](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L230-L274)

In the function `fulfillRandomWords`, the state variable `settings` is re-read at [Line 249](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L249), [Line 250](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L250), and [Line 256](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L256).

To optimize gas savings, consider caching `settings` in a local variable:

```solidity
Settings memory _settings = settings
```

## 3. Unnecessary `msg.sender` cache

Caching `msg.sender` doesn't save gas. In fact, reading from `msg.sender` will cost approximately 2 less gas compared to reading from a local variable. Also, caching `msg.sender` is an unnecessary store operation.

_There are 2 instances of this issue:_

- File: `VRFNFTRandomDrawFactory.sol` [Line 42](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDrawFactory.sol#L42)
- File: `VRFNFTRandomDraw.sol` [Line 279](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L279)

## 4. State variable is being read instead of `memory` variable

In `VRFNFTRandomDraw.sol`, the following `emit InitializedDraw()` is using [`settings`](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L40) (state variable) instead of [`_settings`](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L75) (`memory` variable). As such, reading from a state variable will cost more gas.

File: `VRFNFTRandomDraw.sol` [Line 123](https://github.com/code-423n4/2022-12-forgeries/blob/main/src/VRFNFTRandomDraw.sol#L123)

```solidity
        emit InitializedDraw(msg.sender, settings);
```

As you can see, the emitted event is using `settings` instead of `_settings`.

As such, the code could be refactored to use `_settings` instead of `settings`:

```solidity
        emit InitializedDraw(msg.sender, _settings);
```

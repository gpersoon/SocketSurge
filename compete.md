### Description

The contracts `ExecutionManager`, `TransmitManager`, `SwitchboardBase` and `NativeSwitchboardBase` have both the functions:
`withdrawFees()` and `rescueFunds()`.
These functions both retrieve all the native tokens from the contract and thus compete with eachother.


### Steps to reproduce

1. Call `withdrawFees()` and then call `rescueFunds()`. The second function doesn't retrieve any native token.
1. Or call `rescueFunds()` and then call  `withdrawFees()`. The second function doesn't retrieve any native token.

### Expected behavior
Consider to allow `rescueFunds()` only to rescue ERC20 tokens for contracts where `withdrawFees()` is present.

### Actual behavior

The first function call retrieves all the native tokens.

### Screenshots

### Additional information

[ExecutionManager.sol#L112-L128](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/ExecutionManager.sol#L112-L128), [TransmitManager.sol#L142-L144](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/TransmitManager.sol#L142-L144), [TransmitManager.sol#L243-L249](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/TransmitManager.sol#L243-L249), [SwitchboardBase.sol#L277-L293](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/default-switchboards/SwitchboardBase.sol#L277-L293), [NativeSwitchboardBase.sol#L454-L470](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/native/NativeSwitchboardBase.sol#L454-L470):

```solidity
function withdrawFees(address account_) external onlyRole(WITHDRAW_ROLE) {
    FeesHelper.withdrawFees(account_);
}
function rescueFunds(
    address token_,
    address userAddress_,
    uint256 amount_
) external onlyRole(RESCUE_ROLE) {
    RescueFundsLib.rescueFunds(token_, userAddress_, amount_);
}
```
[FeesHelper.sol#L19-L27](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/libraries/FeesHelper.sol#L19-L27):
```solidity    
function withdrawFees(address account_) internal {
    ...
    uint256 amount = address(this).balance;
    (bool success, ) = account_.call{value: amount}("");
    if (!success) revert TransferFailed();
    ...
}
```

[RescueFundsLib.sol#L25-L40](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/libraries/RescueFundsLib.sol#L25-L40):
```solidity
function rescueFunds(...) internal {
    ...
    if (token_ == ETH_ADDRESS) {
        (bool success, ) = userAddress_.call{value: address(this).balance}("");
        require(success);
    } else {
        ...
    }
}
 ```

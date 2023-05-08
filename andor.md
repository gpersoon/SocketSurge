### Description

The modifier `onlyRemoteSwitchboard()` in `OptimismSwitchboard` contains an invalid combination of the two checks.

This is comparable to issue https://gist.github.com/bytes032/07ca09305cb14d663c5b7efd5f6a92a7 which is classified as high.
So estimated to be high severity due to "Unauthorized access".

### Steps to reproduce

1. Assume `msg.sender == address(crossDomainMessenger__)`
2. Assume `crossDomainMessenger__.xDomainMessageSender() == attacker`
3. Then `msg.sender != address(crossDomainMessenger__)` ==> `false`
4. and `crossDomainMessenger__.xDomainMessageSender() != remoteNativeSwitchboard` ==> `true`
5. The combination of these is `false && true` ==> `false`
6. So the `modifier` won't revert and will allow the `attacker` address

### Expected behavior
The code should be:
```diff
modifier onlyRemoteSwitchboard() override {
    if (
        msg.sender != address(crossDomainMessenger__) 
-        &&
+        ||
        crossDomainMessenger__.xDomainMessageSender() !=
        remoteNativeSwitchboard
    ) revert InvalidSender();
    _;
}
```
### Actual behavior

The current modifier allows any address as `crossDomainMessenger__.xDomainMessageSender()`.
This enables anyone to approve arbitrary roots for arbitrary packets on Optimism native switchboards. 
This breaks the system since switchboards are expected to take on the fraud catching. 
This makes the native switchboard not as secure as native messaging. 

### Screenshots


### Additional information

This is the current code:
[OptimismSwitchboard.sol#L26-L33](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/native/OptimismSwitchboard.sol#L26-L33):

```solidity
modifier onlyRemoteSwitchboard() override {
    if (
        msg.sender != address(crossDomainMessenger__) &&
        crossDomainMessenger__.xDomainMessageSender() !=
        remoteNativeSwitchboard
    ) revert InvalidSender();
    _;
}
```

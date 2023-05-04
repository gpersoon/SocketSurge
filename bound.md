### Description

The function `registerCapacitor()` doesn't verify the value of `maxPacketSize` so it could accidentally or purposely be set 
to an unappropriate/ unusable value for the `Capacitor`.

Specific `Capacitors` might have specific restrictions.
For example if the value is set to `0` then the function `_calculateMinFees()` will revert and the protocol can not operate for the specific Switchboard and Slug, 
because `maxPacketSize` cannot be changed afterwards.

Estimated to have a severity of Medium because it fits in: `Smart contract unable to operate`


### Steps to reproduce

1. Call `registerSwitchBoard()` with an inappropriate value for `maxPacketSize`, for example `0`.


### Expected behavior
Possible solutions:
In function `registerCapacitor()`:
- check that `maxPacketSize_` isn't '0'
- do a call to the `capacitor` to validate the `maxPacketSize`.

Or in function `deploy()` of the `CapacitorFactory` validate the `maxPacketSize`.
It might also be useful to have a permissioned way to update the `maxPacketSize` 

### Actual behavior

The `Socker-DL` protocol might not behave as expected and/or might revert.

Note: also see https://github.com/gpersoon/SocketSurge/blob/main/grief.md 

### Screenshots


### Additional information
Here is the relevant code:

[SwitchboardBase.sol#L129-L140](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/default-switchboards/SwitchboardBase.sol#L129-L140)

```solidity
    function registerCapacitor(
        uint256 siblingChainSlug_,
        address capacitor_,
        uint256 maxPacketSize_
    ) external override {
        if (msg.sender != socket) revert OnlySocket();
        if (isInitialised[siblingChainSlug_]) revert AlreadyInitialised();

        isInitialised[siblingChainSlug_] = true;
        maxPacketSize[siblingChainSlug_] = maxPacketSize_;
        emit CapacitorRegistered(siblingChainSlug_, capacitor_, maxPacketSize_);
    }
```
[SwitchboardBase.sol#L104-L117](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/default-switchboards/SwitchboardBase.sol#L104-L117)
```solidity
function _calculateMinFees(uint32 dstChainSlug_) ... { 
    ...
    switchboardFee =_getMinSwitchboardFees(...) / maxPacketSize[dstChainSlug_]; // this reverts if maxPacketSize == 0
    ...    
}
```
[NativeSwitchboardBase.sol#L298-L311](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/native/NativeSwitchboardBase.sol#L298-L311)

```solidity
 function registerCapacitor(
        uint256,
        address capacitor_,
        uint256 maxPacketSize_
    ) external override {
        if (msg.sender != socket) revert OnlySocket();
        if (isInitialised) revert AlreadyInitialised();

        isInitialised = true;
        maxPacketSize = maxPacketSize_;
        capacitor__ = ICapacitor(capacitor_);

        emit CapacitorRegistered(capacitor_, maxPacketSize_);
    }
```
[NativeSwitchboardBase.sol#L266-L285](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/native/NativeSwitchboardBase.sol#L266-L285)
```solidity
function _calculateMinFees(...) ... {
    ...
    switchboardFee_ = _getMinSwitchboardFees(...) / maxPacketSize; // this reverts if maxPacketSize == 0
    ...
}
```


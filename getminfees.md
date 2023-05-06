### Description

When you call `getMinFees()` before `connect()` has been done, then the `getMinFees()` reverts.
This is not obvious for first time `Plug` designers.

### Steps to reproduce

1. Create a `Plug` and call `getMinFees()`

### Expected behavior
I would expect a return value and if that is not possible a meaningful revert error.

### Actual behavior

Revert without additional information.

### Screenshots



### Additional information

Here is the relevant code with some comments.
[SocketSrc.sol#L128-L173](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/SocketSrc.sol#L128-L173):
```solidity
function getMinFees(
    uint256 msgGasLimit_,
    uint32 remoteChainSlug_,
    address plug_
) external view override returns (uint256 totalFees) {
    PlugConfig storage plugConfig = _plugConfigs[plug_][remoteChainSlug_]; // all 0s if no connect has been done
    ...
    (
        uint256 transmissionFees,
        uint256 switchboardFees,
        uint256 executionFee
    ) = _getMinFees(
            msgGasLimit_,
            remoteChainSlug_,
            plugConfig.outboundSwitchboard__  // is address(0) if no connect has been done
        );
   ...
}
function _getMinFees(..., ISwitchboard switchboard__) ... {
    ...
    (switchboardFees, verificationFee) = switchboard__.getMinFees(remoteChainSlug_); // reverts if no connect has been done
    ...
}
```


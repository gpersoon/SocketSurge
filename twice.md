### Description

The functions `receivePacket()` and `_processMessageFromRoot()` allow receiving the same `packetId_` multiple times.

### Steps to reproduce


### Expected behavior
I would have expected that if a value exists, it can't be overwritten.
If the current behaviour is considered a feature then if would be good to add a comment.

### Actual behavior

The functions `receivePacket()` and `_processMessageFromRoot()` allow receiving the same `packetId_` multiple times.
If the same `packetId_` is send twice then `packetIdToRoot[packetId_]` is overwritten.
Normally the second value for `root_` would be the same as the previous, but if problems would occur elsewhere this might not be the case.

### Screenshots


### Additional information

[NativeSwitchboardBase.sol#L210-L216](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/native/NativeSwitchboardBase.sol#L210-L216):
```solidity
function receivePacket(...) ... {    
    packetIdToRoot[packetId_] = root_; // no check if a value is already present
    ...
}
```

[PolygonL2Switchboard.sol#L112-L123](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/native/PolygonL2Switchboard.sol#L112-L123):
```solidity
function _processMessageFromRoot(...) ... {    
    (bytes32 packetId, bytes32 root) = abi.decode(
        data_,
        (bytes32, bytes32)
    );
    packetIdToRoot[packetId] = root; // no check if a value is already present
    ...
}
```    

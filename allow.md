### Description

The function `allowPacket()` of `NativeSwitchboards` doesn't catch invalid `packetId`s.

Estimated to have a severity of Low because other parts of the code catch the invalid `packetId`s.


### Steps to reproduce

1. Assume a `NativeSwitchboard` is being used
2. Call function `execute()` with an invalid `packetId`
3. Function `execute()` calls  `_verify()` with the invalid `packetId`
4. `packetIdRoots[packetId_]` will be `0`
5. Function `_verify()` calls `allowPacket()` with the parameters: `root_ == 0` and `packetId_` is invalid
6. In function `allowPacket()` then `packetIdToRoot[packetId_]` will be `0`
7. So function `allowPacket()` returns `true`
8. Function `_verify()` continues executing while the `packetId` should not have been allowed.
9. Luckily the next steps will catch the invalid `packetId`

### Expected behavior
Possible solutions:
```diff
function allowPacket(bytes32 root_,bytes32 packetId_,...) ... {
    ...
+   if (packetIdToRoot[packetId_] == bytes32(0) ) return false;
    if (packetIdToRoot[packetId_] != root_) return false;
    return true;
}
```

### Actual behavior

Function `_verify()` continues executing while the `packetId` should not have been allowed.

### Screenshots

### Additional information

[SocketDst.sol#L155-L178](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/SocketDst.sol#L155-L178):

```solidity
function _verify(bytes32 packetId_, ... ) ... {    
    if (
        !ISwitchboard(plugConfig_.inboundSwitchboard__).allowPacket(
            packetIdRoots[packetId_],
            packetId_,
            uint32(remoteChainSlug_),
            rootProposedAt[packetId_]
        )
    ) revert VerificationFailed();
    ...
}
```

[NativeSwitchboardBase.sol#L224-L234](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/native/NativeSwitchboardBase.sol#L224-L234):
```solidity
function allowPacket(bytes32 root_,bytes32 packetId_,...) ... {
    ...
    if (packetIdToRoot[packetId_] != root_) return false;
    return true;
}
```

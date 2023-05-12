### Description

The validity of `packetId` isn't checked enough. In combination with an error in a `Decapacitor` this could 
lead to unauthorized execution of messages.
Note: the current code doesn't have this problem, but future updates / future `Decapacitors` could introduce a bug.

Estimated to have a severity of Medium because it fits in: `Unauthorised Access`, but it is unlikely to happen.


### Steps to reproduce

1. Call function `execute()` with a `packetId` that hasn't been proposed.
2. This calls `_verify` with a `packetId` that hasn't been proposed.
3. Function `_verify` now `packetIdRoots[packetId_]==0` but this isn't detected here.
4. Function `_verify` calls `allowPacket()` with `root_==0`. This is not detected, see issue https://github.com/gpersoon/SocketSurge/blob/main/allow.md
5. Function `_verify` calls `verifyMessageInclusion()` with `root_==0`
6. This might not be detected if there is an error in the `Decapacitor`.


### Expected behavior
A solution would be to add something like the following in function `execute()`:
```solidity
if (packetIdRoots[packetId_] == 0) revert NotProposed();
```

### Actual behavior

An error in a `decapacitor` would mistakenly allow the executing of an invalid message.

### Screenshots


### Additional information

[SocketDst.sol#L155-L178](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/SocketDst.sol#L155-L178):

```solidity
function _verify(...) ... {
    if (
        !ISwitchboard(plugConfig_.inboundSwitchboard__).allowPacket(
            packetIdRoots[packetId_],
            packetId_,
            uint32(remoteChainSlug_),
            rootProposedAt[packetId_]
        )
    ) revert VerificationFailed();

    if (
        !plugConfig_.decapacitor__.verifyMessageInclusion(
            packetIdRoots[packetId_],
            packedMessage_,
            decapacitorProof_
        )
    ) revert InvalidProof();
}
```
    

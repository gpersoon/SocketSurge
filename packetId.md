### Description

Misconfigured `Socket` plus misconfigured `Transmitter` could lead to to duplicate `PacketId`s.
Set to severity of low because it could fit in `Smart contract unable to operate` or `Damage to users/protocol due to griefing`, 
but is unlikely to happen.

### Steps to reproduce

1. (Accidentally) deploy a `Socket` contract on a new chain with the same `Slug` as another chain
2. or deploy a second version of a `Socket` contract on the same chain
3. (Accidentally) allow a `Transmitter` to also process (e.g. Seal) this `Socket`
4. Or do this on purpose if `Transmitter`s would be permissionless
5. Assume the same `capacitorAddress` is used (this can happen if a deterministic deployment of Switchboards is used)
6. Then duplicate `PacketId` will be created because it is build of local `Slug`, local `capacitorAddress` and local `packetCount`
7. Assume the FastSwitchboard is used
8. Do an `attest()` on the destionation chain with the duplicate `PacketId`
9. Now addition `attest()`s are done and a message could be accepted that maybe should not have been accepted


### Expected behavior
Consider storing more information in `PacketId`, for example make it a hash and include the `root`. 
This would be more robust.

### Actual behavior

The function `attest()` keeps record of attested messages via `PacketId`.
The `PacketId` is build of local `Slug`, local `capacitorAddress` and local `packetCount`, so only stores general information
from the source chain.

### Screenshots


### Additional information

[FastSwitchboard.sol#L67-L88](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/default-switchboards/FastSwitchboard.sol#L67-L88):
 
```solidity
function attest(bytes32 packetId_,...) ... {
    ...
    if (isAttested[watcher][packetId_]) revert AlreadyAttested();
    ...
    isAttested[watcher][packetId_] = true;
    ...
}
```

[SocketSrc.sol#L210-L220](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/SocketSrc.sol#L210-L220):
```solidity
function _encodePacketId(...) ... {   
    return
        bytes32(
            (uint256(chainSlug) << 224) |
                (uint256(uint160(capacitorAddress_)) << 64) |
                packetCount_
        );
}
```

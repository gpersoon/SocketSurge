### Description

Misconfigured `Socket` plus misconfigured `Transmitter` could lead to duplicate `MsgId`s.
Set to severity of low because it could fit in `Smart contract unable to operate` or `Damage to users/protocol due to griefing`, 
but is unlikely to happen.

### Steps to reproduce

1. (Accidentally) deploy a `Socket` contract on a new chain with the same `Slug` as another chain
2. or deploy a second version of a `Socket` contract on the same chain
3. (Accidentally) allow a `Transmitter` to also process (e.g. Seal) this `Socket`
4. Or do this on purpose if `Transmitter`s would be permissionless
5. Then a duplicate `MsgId` will be created, because `MsgId` is build of local `Slug` and local `messageCount`.
6. Do an `execute()` on the destionation chain with the duplicate `MsgId`
7. If it is slower than the original `MsgId` then it will be rejected.
8. If it is faster than the original `MsgId` then it will be processed and the original `MsgId` will be rejected.
9. This would lead to DOS / Griefing

### Expected behavior
Consider using `packedMessage` as the parameter for `messageExecuted[]`.
This would be more robust.

### Actual behavior

The function `execute()` keeps record of processed messages via `MsgId`.
The `MsgId` is build of local `Slug` and local `messageCount`, so only stores general information
from the source chain.

### Screenshots


### Additional information
[SocketDst.sol#L108-L153](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/SocketDst.sol#L108-L153):
```solidity
function execute(...) ... {
    if (messageExecuted[messageDetails_.msgId])
        revert MessageAlreadyExecuted();
    messageExecuted[messageDetails_.msgId] = true;
    ...    
}
```
[SocketSrc.sol#L206-L208](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/SocketSrc.sol#L206-L208):
```solidity
function _encodeMsgId(uint256 slug_) internal returns (bytes32) {
    return bytes32((uint256(uint32(slug_)) << 224) | messageCount++);
}
```

### Description

When using `HashChainCapacitor`, then `seal()` has to be perfectly synchronized with `_chainLength` of `addPackedMessage()` .
If `seal()` is called too early it processes partial information and the rest of the information, 
until `_MAX_LEN` is reached, is never processed.

Estimated to have a severity of Medium because it fits in: `Smart contract unable to operate`.


### Steps to reproduce

1. Add one packet via `outbound()` and thus `addPackedMessage()`. 
   Afterwards `_nextPacketCount==0`, `_chainLength==1` and `_roots[0]` is filled
2. The transmitter now calls `seal()`, which calls `sealPacket()`.
   Afterwards `_nextSealCount==1`
3. Add a second packet via `outbound()` and thus `addPackedMessage()`. 
   Afterwards `_nextPacketCount==0`, `_chainLength==2` and `_roots[0]` is updated
4. The transmitter now calls `seal()`, which calls `sealPacket()`. `_nextSealCount==1` and `roots[0]==0` so it reverts
5. Now add packets repeatedly via `outbound()` until `_chainLength == _MAX_LEN` and `_nextPacketCount` is increased
   Afterwards `_nextPacketCount==1`, `_chainLength==0` and `_roots[0]` is updated
6. Add a next packet via `outbound()` and thus `addPackedMessage()`. 
   Afterwards `_nextPacketCount==1`, `_chainLength==1` and `_roots[1]` is filled so `seal()` can be used again.
7. The transmitter now calls `seal()`, which calls `sealPacket(). 
   Now `_nextSealCount==1` and `roots[1]` is filled so `sealPacket()` returns `roots[1]` to be processed.
   However all the information that was stored in `roots[0]` (e.g. the packet 2 to 10) is ignored.
   

### Expected behavior

A possible solution is store the `root` values in temporary variable and only assign it to `_roots[]` when it is ready.
See example below:

```diff
+   uint tmpRoot;
    function addPackedMessage(bytes32 packedMessage_) external override onlySocket {
        uint64 packetCount = _nextPacketCount;

-       _roots[packetCount] = keccak256(abi.encode(_roots[packetCount], packedMessage_));
+       tmpRoot = keccak256(abi.encode(tmpRoot, packedMessage_));
        _chainLength++;

        if (_chainLength == _MAX_LEN) {
            _nextPacketCount++;
            _chainLength = 0;
+           _roots[packetCount] = tmpRoot;
+           tmpRoot = 0;
        }

        emit MessageAdded(packedMessage_, packetCount, _roots[packetCount]);
    }
```


### Actual behavior

It only works if `seal()` is done after the `roots[]` are completely filled (e.g. with `_MAX_LEN` packets).
However for the Transmitter its not easy to know that the `roots[]` are completely filled:
Both `_chainLength` and `_MAX_LEN` are private.
So in practice it is probably not synchronized.

### Screenshots


### Additional information

[HashChainCapacitor.sol#L36-L52](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/capacitors/HashChainCapacitor.sol#L36-L52)

```solidity
    function addPackedMessage(
        bytes32 packedMessage_
    ) external override onlySocket {
        uint64 packetCount = _nextPacketCount;

        _roots[packetCount] = keccak256(
            abi.encode(_roots[packetCount], packedMessage_)
        );
        _chainLength++;

        if (_chainLength == _MAX_LEN) {
            _nextPacketCount++;
            _chainLength = 0;
        }

        emit MessageAdded(packedMessage_, packetCount, _roots[packetCount]);
    }
```
	
[BaseCapacitor.sol#L52-L60](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/capacitors/BaseCapacitor.sol#L52-L60)
```solidity
    function sealPacket(
        uint256
    ) external virtual override onlySocket returns (bytes32, uint64) {
        uint64 packetCount = _nextSealCount++;
        if (_roots[packetCount] == bytes32(0)) revert NoPendingPacket();

        bytes32 root = _roots[packetCount];
        return (root, packetCount);
    }
```


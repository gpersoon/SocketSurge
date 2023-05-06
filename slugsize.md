### Description

Slugs are represented sometimes by `uint32` and sometimes by `uint256`.
Slugs are normally less or equal to `type(uint32).max` so it will fit in both the types.

The different types could potentially lead to errors if too large values are used for `Slug` (e.g larger than `type(uint32).max`).
Especially in combination with the construction `uint32(Slug)`. This construction truncates the `Slug` without error.
This construction occurs a few time in the code, see below.

Luckily in the current code there is no issue, however this might change with future updates.
It is safer and more consistent to use `uint32` everywhere.

### Steps to reproduce


### Expected behavior
Use the same type everywhere.

### Actual behavior

In current code no issue but potenial issue in future updates.

### Screenshots

-

### Additional information

[SocketSrc.sol#L35-L75](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/SocketSrc.sol#L35-L75):

```solidity
function outbound(uint256 remoteChainSlug_, ... ) ... {
    ...
    ISocket.Fees memory fees = _deductFees(
        msgGasLimit_,
        uint32(remoteChainSlug_), // potential truncate
        plugConfig.outboundSwitchboard__
    );
   ...    
}
```
[SocketDst.sol#L80-L100](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/SocketDst.sol#L80-L100):

```solidity
function propose(...) ... {
    ...
    (address transmitter, bool isTransmitter) = transmitManager__
        .checkTransmitter(
            uint32(_decodeSlug(packetId_)), // potential truncate (_decodeSlug returns uint256)
            keccak256(abi.encode(chainSlug, packetId_, root_)),
            signature_
        );
    ...
    
}
```
[SocketDst.sol#L226-L230](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/SocketDst.sol#L226-L230):

```solidity
    function _decodeSlug(
    bytes32 id_
) internal pure returns (uint256 chainSlug_) { // could return uint32
    chainSlug_ = uint256(id_) >> 224;
}
```

[SocketDst.sol#L155-L178](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/SocketDst.sol#L155-L178):

```solidity
function _verify(..., uint256 remoteChainSlug_, ...) internal view {
    if (
        !ISwitchboard(plugConfig_.inboundSwitchboard__).allowPacket(
            packetIdRoots[packetId_],
            packetId_,
            uint32(remoteChainSlug_), // potential truncate
            rootProposedAt[packetId_]
        )
    ) revert VerificationFailed();
    ...
}
```


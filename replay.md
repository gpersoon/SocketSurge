### Description

Function `propose()` can be called with a previousely used signature of `setSourceGasPrice()` (e.g. replay) because the signed data in `propose()` and `setSourceGasPrice()` is very similar. 
Luckily this has no negative effects for the protocol so the severity is set to low.
However future updates might make this feasable.

### Steps to reproduce
1. Call `propose()` with the signature of `setSourceGasPrice()`

### Expected behavior
I would expect `setSourceGasPrice()` as well as `setRelativeGasPrice()` to add a string first, like several other functions do.

```diff
function setSourceGasPrice(...) ... {
    (address transmitter, bool isTransmitter) = transmitManager__
        .checkTransmitter(
            chainSlug,
-            keccak256(abi.encode(chainSlug, nonce_, sourceGasPrice_)),
+            keccak256(abi.encode("SET_SOURCE_GAS_PRICE",chainSlug, nonce_, sourceGasPrice_)),            
            signature_
        );
    ...
}
function setRelativeGasPrice(...) ... {
    (address transmitter, bool isTransmitter) = transmitManager__
        .checkTransmitter(
            siblingChainSlug_,
-            keccak256(abi.encode(chainSlug,siblingChainSlug_,nonce_,relativeGasPrice_)
+            keccak256(abi.encode("SET_RELATIVE_GAS_PRICE",chainSlug,siblingChainSlug_,nonce_,relativeGasPrice_)
            ),
            signature_
        );
}
```

### Actual behavior
The `keccak256()`s of functions `propose()` and `setSourceGasPrice()` can result in the same value as shown in the POC below.

```solidity
// SPDX-License-Identifier: none
pragma solidity >=0.8.19;
import "hardhat/console.sol"; 

contract Test {
    constructor () {
    uint32  chainSlug = 1234;
    bytes32 packetId ="Test";
    bytes32 root = "Root";
    uint256 nonce = uint(packetId);
    uint256 sourceGasPrice=uint(root);
    console.logBytes32(keccak256(abi.encode(chainSlug, packetId, root)));
        // 0x575611a737e21649b8336e29fa99a73ce53255330580a38c249eb5a2aab5b84e 
    console.logBytes32(keccak256(abi.encode(chainSlug, nonce, sourceGasPrice)));
        // 0x575611a737e21649b8336e29fa99a73ce53255330580a38c249eb5a2aab5b84e
    }
}
```
Function `propose()` will set a (`root,`packedId`) will correspond to a (`nonce`,`sourceGasPrice`).
However as these values are invalid they will not do any harm.
The only side effect is some extra data in `packetIdRoots` and `rootProposedAt` as well as an extra `emit`.

Note: Function `setSourceGasPrice()` will not accept these values from `propose()` because `packetId_` of `propose()` includes the `capacitorAddress` so in practice it will never overlap with a valid `nonce`.

### Screenshots

### Additional information
[SocketDst.sol#L80-L100](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/SocketDst.sol#L80-L100):
```solidity
uint32 public immutable chainSlug; // defined in SocketBase
function propose(bytes32 packetId_,bytes32 root_,...) ... {
    ...
    (address transmitter, bool isTransmitter) = transmitManager__
        .checkTransmitter(
            uint32(_decodeSlug(packetId_)),
            keccak256(abi.encode(chainSlug, packetId_, root_)),
            signature_
        );
    if (!isTransmitter) revert InvalidAttester();
    ...
}
```

[GasPriceOracle.sol#L88-L109](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/GasPriceOracle.sol#L88-L109):
```solidity
uint32 public immutable chainSlug;
function setSourceGasPrice(uint256 nonce_,uint256 sourceGasPrice_,...) ... {
    (address transmitter, bool isTransmitter) = transmitManager__
        .checkTransmitter(
            chainSlug,
            keccak256(abi.encode(chainSlug, nonce_, sourceGasPrice_)),
            signature_
        );
    if (!isTransmitter) revert TransmitterNotFound();
    ...    
}
```


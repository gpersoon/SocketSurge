**Context:**  [SwitchboardBase.sol](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/default-switchboards/SwitchboardBase.sol), [AccessRoles.sol](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/utils/AccessRoles.sol), [NativeSwitchboardBase.sol](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/native/NativeSwitchboardBase.sol)

**Description:**
The contract `SwitchboardBase` uses two different definitions of the `GAS_LIMIT_UPDATER_ROLE`.

- In `AccessRoles.sol` it is defined as: `keccak256("GAS_LIMIT_UPDATER_ROLE")`.
- In function `setExecutionOverhead` of `SwitchboardBase` it is used as `"GAS_LIMIT_UPDATER_ROLE"` (so without the `keccak256()`).
- However in function `setExecutionOverhead` of `NativeSwitchboardBase` it is used as `GAS_LIMIT_UPDATER_ROLE` (so with the `keccak256()`).

This is confusing and can lead to configuration mistakes.
There are other roles as well that are string based, without `keccak256()`:
- "TRANSMITTER_ROLE"
- "WATCHER_ROLE"
- "TRIP_ROLE"
- "UNTRIP_ROLE"

Below is some code that shows the issue:
In contract `AccessRoles.sol`:

```solidity
bytes32 constant GAS_LIMIT_UPDATER_ROLE = keccak256("GAS_LIMIT_UPDATER_ROLE");
```

In contract `SwitchboardBase.sol`:

```solidity
import {GOVERNANCE_ROLE, WITHDRAW_ROLE, RESCUE_ROLE, GAS_LIMIT_UPDATER_ROLE} from "../../utils/AccessRoles.sol";

abstract contract SwitchboardBase is ISwitchboard, AccessControlExtended {
    ...
    function setExecutionOverhead(...) ... {
        ...
        if (!_hasRole("GAS_LIMIT_UPDATER_ROLE", dstChainSlug_, gasLimitUpdater))
            revert NoPermit("GAS_LIMIT_UPDATER_ROLE");
        ...      
    }
    ...
}
```
In contract `NativeSwitchboardBase.sol`:
```solidity
import {GAS_LIMIT_UPDATER_ROLE, GOVERNANCE_ROLE, RESCUE_ROLE, WITHDRAW_ROLE, TRIP_ROLE, UNTRIP_ROLE} from "../../utils/AccessRoles.sol";
 
abstract contract NativeSwitchboardBase is ISwitchboard, AccessControlExtended {
    ...
    function setExecutionOverhead(...) ... {
        ...        
        if (!_hasRole(GAS_LIMIT_UPDATER_ROLE, gasLimitUpdater))
            revert NoPermit(GAS_LIMIT_UPDATER_ROLE);
    }    
    ...
}
```

**Recommendation:**
 Check all the roles and preferable always use the `keccak256()`ed version.
 

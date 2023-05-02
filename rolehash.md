**Context:**  [SwitchboardBase.sol](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/default-switchboards/SwitchboardBase.sol), [AccessRoles.sol](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/utils/AccessRoles.sol), [NativeSwitchboardBase.sol](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/native/NativeSwitchboardBase.sol)

**Severity:** Low

**Description:**
The contracts `SwitchboardBase` and `NativeSwitchboardBase` uses two different definitions of `TRIP_ROLE` and `UNTRIP_ROLE`.

- In `AccessRoles.sol` the values are defined as: `keccak256("TRIP_ROLE")` and `keccak256("UNTRIP_ROLE")`.
- In `SwitchboardBase` it is used as `"TRIP_ROLE"` and `"UNTRIP_ROLE"` (so without the `keccak256()`).
- However in `NativeSwitchboardBase` it is used as `TRIP_ROLE` and `UNTRIP_ROLE` (so with the `keccak256()`).

This means that an account that want to `TRIP`/`UNTRIP` multiple Switchboards, needs to have seperate autorizations, depending on which version of Switchboard is being used. This is non consistent, confusing and errorphrone.

Below is some code to show the difference of the the version with and without `keccak256()`.
```solidity
// SPDX-License-Identifier: none
pragma solidity >=0.8.19;
import "hardhat/console.sol"; 

contract TestRole {
    constructor() {
        hasRole("test"); // 0x7465737400000000000000000000000000000000000000000000000000000000
        hasRole(keccak256("test")); // 0x9c22ff5f21f0b81b113e63f7db6da94fedef11b2119b4088b89664fb9a3cb658
    }
    function hasRole(bytes32 role_) public view  {
        console.logBytes32(role_);
    }
}
```

Below is some code that shows where the issue in the source code:
In contract `AccessRoles.sol`:
```solidity
bytes32 constant TRIP_ROLE = keccak256("TRIP_ROLE");
bytes32 constant UNTRIP_ROLE = keccak256("UNTRIP_ROLE");
```

In contract `SwitchboardBase.sol`:

```solidity
abstract contract SwitchboardBase is ISwitchboard, AccessControlExtended {
    ...
    function tripGlobal(uint256 nonce_, bytes memory signature_) external {
        ...
        if (!_hasRole("TRIP_ROLE", tripper)) revert NoPermit("TRIP_ROLE");
        ...
    }
     function untrip(uint256 nonce_, bytes memory signature_) external {
        ...
        if (!_hasRole("UNTRIP_ROLE", untripper)) revert NoPermit("UNTRIP_ROLE");
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
    function tripGlobal(uint256 nonce_, bytes memory signature_) external {
        ...
        if (!_hasRole(TRIP_ROLE, watcher)) revert NoPermit(TRIP_ROLE);
        ...
    }
    function untrip(uint256 nonce_, bytes memory signature_) external {
        ...
        if (!_hasRole(UNTRIP_ROLE, watcher)) revert NoPermit(UNTRIP_ROLE);
        ...
    }
    ...
}
```

**Recommendation:**
Always use the `keccak256()`ed version.

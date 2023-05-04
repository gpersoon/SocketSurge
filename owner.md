**Title:** `Owner` can manipulate `totalWatchers[]` in `FastSwitchboard`.

**Context:**  [FastSwitchboard.sol#L156-L180](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/switchboard/default-switchboards/FastSwitchboard.sol#L156-L180), [AccessControl.sol#L54-L59](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/utils/AccessControl.sol#L54-L59)

**Severity:** Low

**Description:**
 An `Owner` can manipulate `totalWatchers[]` in `FastSwitchboard` because it can do `grantRole()` and `revokeRole()` directly.
 
 The functions `grantWatcherRole()` and `revokeWatcherRole()` keep a score of the number of `watchers` 
 in the array `totalWatchers[]`. However an `owner` can directly do `grantRole()` and `revokeRole()`, 
 which doesn't update the array `totalWatchers[]`.
 
 This way the owner could artifically lower or higher the values in `totalWatchers[]`:
 - first assign himself the right for `GOVERNANCE_ROLE` via `grantRole()`;
 - then do repeatedly `grantWatcherRole()` and `revokeRole()`;
 - or do repeatedly `grantRole()` and `revokeWatcherRole()`.
 
A lower value of `totalWatchers[]` means less watchers are required to `attest()`.
A higher value of `totalWatchers[]` means `attest()` can never set `isPacketValid[]` to be `true`.
 
A piece of `FastSwitchboard.sol`:
```solidity
    function grantWatcherRole(
        uint256 srcChainSlug_,
        address watcher_
    ) external onlyRole(GOVERNANCE_ROLE) {
        if (_hasRole("WATCHER_ROLE", srcChainSlug_, watcher_))
            revert WatcherFound();
        _grantRole("WATCHER_ROLE", srcChainSlug_, watcher_);

        totalWatchers[srcChainSlug_]++;
    }

    function revokeWatcherRole(
        uint256 srcChainSlug_,
        address watcher_
    ) external onlyRole(GOVERNANCE_ROLE) {
        if (!_hasRole("WATCHER_ROLE", srcChainSlug_, watcher_))
            revert WatcherNotFound();
        _revokeRole("WATCHER_ROLE", srcChainSlug_, watcher_);

        totalWatchers[srcChainSlug_]--;
    }
```

**Recommendation:**
Consider keeping track of the number of owners of a role in contract `AccessControl`, in the functions `grantRole()` and `revokeRole()`.

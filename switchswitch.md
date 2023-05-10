### Description

Switchboards can be changed along the way.
Depending on the implementation of the switchboard this can allow the passing of messages that would normally not be allowed.

Not sure how to estimate severity.

### Steps to reproduce

1. Assume a connection has been setup via `FastSwitchboard` 
2. Assume insufficient Watchers have attested so `allowPacket()` is still `false`
3. The receiver calls `connect()` again, now via `OptimisticSwitchboard`
4. Assume sufficient time has passed
5. Now `allowPacket()` from `OptimisticSwitchboard` is `true` and `execute()` can be performed.

### Expected behavior
I would expect that the same type of switchboard should be present on both sides and that this is somehow enforced.

### Actual behavior

By changing switchboards along the way packets might be delivered that normally would not have been delivered.
Additionally different switchboards might have different fees and this perhaps allows for a lower fee than normally required.

### Screenshots


### Additional information

Add any other context about the problem here.

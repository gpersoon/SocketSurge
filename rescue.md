### Description

Most of the contracts have a `rescueFunds()` function that can rescue any funds and tokens that are accidentally send to the contract.
The one exception is contract `Hasher`.

`Hasher` doesn't have any `payable` functions to ETH would not normally be present in the contract.
Tokens could be send to this contract, but that doesn't seem very likely.

Adding a function `rescueFunds()` does increase the complexity so probably not worth the trouble to implement.
However adding a comment in the contract might be useful (also to prevent future remaks about this).

### Steps to reproduce


### Expected behavior
Would expect `Hasher` to also have this function or have a comment why it is lacking.

### Actual behavior

Tokens send to the `Hasher` contract can't be rescued.

### Screenshots

### Additional information

[Hasher.sol#L12](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/utils/Hasher.sol#L12):
```solidity
contract Hasher is IHasher {

    function packMessage(...) external pure override returns (bytes32) {
        ...
    }
    // no rescueFunds
}
```

[Socket.sol#L44-L50](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/socket/Socket.sol#L44-L50):
```solidity
function rescueFunds(
    address token_,
    address userAddress_,
    uint256 amount_
) external onlyRole(RESCUE_ROLE) {
    RescueFundsLib.rescueFunds(token_, userAddress_, amount_);
}
```

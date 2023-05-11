### Description

The function `rescueFunds()` ignores `amount` for native tokens.


### Steps to reproduce

1. Call `rescueFunds(ETH_ADDRESS,userAddress,1e100 ETH)`


### Expected behavior
I would expect that only `amount` of native token is transferred as described in the comments.

### Actual behavior

The function `rescueFunds()` is supposed the rescue a certain `amount` of tokens.
However when recueing the native token then the entire `balance` is transferred.
The `amount` is ignored in that case.

### Screenshots

 

### Additional information

[RescueFundsLib.sol#L19-L40](https://github.com/SocketDotTech/socket-DL/blob/master/contracts/libraries/RescueFundsLib.sol#L19-L40):

```solidity
/**
 * @dev Rescues funds from a contract.
 * @param token_ The address of the token contract.
 * @param userAddress_ The address of the user.
 * @param amount_ The amount of tokens to be rescued.
 */
function rescueFunds(
    address token_,
    address userAddress_,
    uint256 amount_
) internal {
    require(userAddress_ != address(0));

    if (token_ == ETH_ADDRESS) {
        (bool success, ) = userAddress_.call{value: address(this).balance}( // entire balance
            ""
        );
        require(success);
    } else {
        IERC20(token_).transfer(userAddress_, amount_);
    }
}
```    

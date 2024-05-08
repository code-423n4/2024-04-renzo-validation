### [L-01] L2 Oracles can be arbitraged to get the best ezETH value

All oracles seem to use 0x00 as their oracle address right now (means manually updated price), which can be dangerous because it is not always synced with the actual price of ezETH. Users can use this stale data in L2 to profit from the difference in price between the ezETH value from L2 and L1.

It'll be good to start using the Chainlink oracles (ezETH / ETH) for L2s

### [L-02] Chainlink has no ezETH / ETH oracle for Binance chain

In due time, the L2s will query the value of ezETH through Chainlink, and some L2s like Base and Linea has the ezETH/ETH Chainlink oracle set up. However, binance chain still does not have the oracles, and it is one of the chains that Renzo supposedly integrates with.

https://data.chain.link/feeds/scroll/mainnet/ezeth-eth

### [L-03] payable.transfer is used to transfer ETH

payable.transfer may not be compatible with smart contract recipients. Specifically, the transfer will inevitably fail when the smart contract:

- does not implement a payable fallback function, or
- implements a payable fallback function which would incur more than 2300 gas units, or
- implements a payable fallback function incurring less than 2300 gas units but is called through a proxy that raises the call’s gas usage above 2300.

```
        // send selected redeem asset to user
        if (_withdrawRequest.collateralToken == IS_NATIVE) {
            payable(msg.sender).transfer(_withdrawRequest.amountToRedeem);
```

use payable.call instead.

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L301C1-L303C75

### [L-04] Users can block the withdraw by not withdrawing their tokens

When withdrawing tokens, `claimReserve` increases as part of the check for the withdraw buffer. To reduce the value of `claimReserve`, `claim()` has to be called.

```
    function claim(uint256 withdrawRequestIndex) external nonReentrant {
        // check if provided withdrawRequest Index is valid
        if (withdrawRequestIndex >= withdrawRequests[msg.sender].length)
            revert InvalidWithdrawIndex();

        WithdrawRequest memory _withdrawRequest = withdrawRequests[msg.sender][
            withdrawRequestIndex
        ];
        if (block.timestamp - _withdrawRequest.createdAt < coolDownPeriod) revert EarlyClaim();

        // subtract value from claim reserve for claim asset
->      claimReserve[_withdrawRequest.collateralToken] -= _withdrawRequest.amountToRedeem;
```

Users can grief the system by simply not calling `claim()`. Other users cannot withdraw their tokens since there will be no token available to withdraw.

The owner then has to increase the withdraw buffer limit as a work around.

It will be good if the protocol can somehow force the user to `claim()` if the user does not `claim()` their tokens in due time, so that there is no need to continue updating the withdraw buffer and prevent any griefing cases.

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L279-L311

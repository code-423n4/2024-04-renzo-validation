## [L-01] ERC20's that do not have a decimals() function are not supported by the protocol.

In RestakeManager, addCollateralToken():

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L220
```js
    if (IERC20Metadata(address(_newCollateralToken)).decimals() != 18)
        revert InvalidTokenDecimals(
            18,
            IERC20Metadata(address(_newCollateralToken)).decimals()
        );
```
However, while widely used, the decimals() function is an optional part of the [ERC20 standard](https://eips.ethereum.org/EIPS/eip-20), thus some weird tokens may be incompatible.

# Informational:

## [I-01] Withdrawal Request nonce starts from 1, not 0
The withdrawal request nonce starts from 1 and not 0. This may impact other protocols interacting with Renzo that rely on tracking withdrawRequestNonces. It is not clear that it starts from 1, it is best to have increment it after pushing to the array, or clearly document that in the function to prevent developers from messing it up.

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L239
```js
        withdrawRequestNonce++;

        // add withdraw request for msg.sender
        withdrawRequests[msg.sender].push(
            WithdrawRequest(
                _assetOut,
                withdrawRequestNonce,
                amountToRedeem,
                _amount,
                block.timestamp
            )
        );
```

## [I-02] Typo in inflationPercentaage in RenzoOracle

Should be inflationPercentaage, not [inflationPercentaage](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L135C17-L135C38).
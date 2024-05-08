## Description

When `xRenzoDeposit::getMintRate()` is called, the function checks if the `reciever` and the `oracle` are set, however the function reverts only when both variables are not set

```solidity
if (receiver == address(0) && address(oracle) == address(0)) revert PriceFeedNotAvailable();
```

## SUGGESTION

Consider reverting when either of the variables are not set and modify the function as shown below

```solidity
if (receiver == address(0) || address(oracle) == address(0)) revert PriceFeedNotAvailable();
```
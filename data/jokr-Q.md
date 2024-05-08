## [L-01] safeApprove function is deprecated

Some non-standard tokens like will revert when a contract or a user tries to approve an allowance when the spender allowance has already been set to a non zero value. In the current code we don't have any problem because allowance given using is always consumed and will be set to 0. However, if the approval is not lowered to exactly 0 (due to a rounding error or another unfore-
seen situation) then the next approval using safeApprove will fail (assuming a token like USDT is used), blocking all further deposits.

We also should note that OpenZeppelin has officially deprecated the safeApprove function, suggesting to use instead safeIncreaseAllowance and safeDecreaseAllowance


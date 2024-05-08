# [1-L] misleading revert error
in [`LockboxAdapterBlast::bridgeTo()`](https://github.com/code-423n4/2024-04-renzo/blob/1c7cc4e632564349b204b4b5e5f494c9b0bc631d/contracts\Bridge\Connext\integration\LockboxAdapterBlast.sol#L56-L67) it has `revert AmountLessThanZero();` but actually it checks for `=< 0` so better name the error `invalid amount`

# [2-L] in consistent natspec with function
in those [lines](https://github.com/code-423n4/2024-04-renzo/blob/1c7cc4e632564349b204b4b5e5f494c9b0bc631d/contracts\Bridge\L2\PriceFeed\ConnextReceiver.sol#L110-L123) natspec says the opposite of the function implementation leading to misleading of leader or adminUser

# [3-L] absent access control
implement access control to be callable only by OD [here](https://github.com/code-423n4/2024-04-renzo/blob/1c7cc4e632564349b204b4b5e5f494c9b0bc631d/contracts\Deposits\DepositQueue.sol#L152-L156) to prevent users from loss of funds 

# [4-L] in consistent natspec with function
in those [lines](https://github.com/code-423n4/2024-04-renzo/blob/1c7cc4e632564349b204b4b5e5f494c9b0bc631d/contracts\Bridge\L2\PriceFeed\CCIPReceiver.sol#L117-L127) natspec says the opposite of the function implementation leading to misleading of leader or adminUser
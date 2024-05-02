# [L-01] Admin related function should emit events
Admin setter functions should emit events for user covinience
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L121
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L215
## Recommendation
Add proper events for these specific functions

# [L-02] Use of deprecated function `safeApprove`
OZ `safeApprove` is now a deprecated fucntion , so using it would result in error
Below are the some of the instances where safeApprove is used
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L552
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L559
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol#L142
## Recommendation 
Use safeIncreaseAllowance or safeDecreaseAllowance instead

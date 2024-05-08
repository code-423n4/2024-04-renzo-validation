According to the comments in the document, `getTotalRewardsEarned` in `RestakeManager` should return all revenue generated within the protocol. 

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L670-L707

However, the admin has the authority to add or remove collateral, but this function does not account for the revenue generated from collateral that has already been removed.

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L243-L259


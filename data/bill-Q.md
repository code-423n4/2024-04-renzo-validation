# The function `getTotalRewardsEarned` does not account for removed collateral tokens.
According to the comments in the document, `getTotalRewardsEarned` in `RestakeManager` should return all revenue generated within the protocol. 

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L670-L707

However, the admin has the authority to add or remove collateral, but this function does not account for the revenue generated from collateral that has already been removed.

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L243-L259

# The stETH transferred amount may be less than the requested amount
The stETH transferred amount may be less than the requested amount due to the use of shares for accounting purposes. This can lead to precision loss during the conversion between the amount transferred and the number of shares, resulting in the actual amount of stETH transferred being lower than expected.

For example, transferring 2 units of stETH results in only 1 unit being credited to the protocol.

The protocol does not check the actual amount transferred, which could lead to an error in calculating the amount of ezETH to be minted, mistakenly overestimating the amount of stETH deposited by one unit.

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L539-L540







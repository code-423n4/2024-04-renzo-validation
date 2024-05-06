# [L-01] Admin related function should emit events
Some Admin setter functions does not emit events 
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L121
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L215
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol#L101
## Recommendation
Add proper events for these specific functions

# [L-02]
Due to the nature of the Chainlink price oracles, the upcoming price updates are predictable. Additionally, the predictable price update can be forced by an attacker by calling the `updatePrice` for xRENZODeposit function, which is restricted, which mitigates the arbitrage opportunity a bit. SO depending upon the admin It allows the them to predict a profitable price deviation and perform a deposit, external DEX swap, or withdraw to get a guaranteed profit from their actions 
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol#L320
If the protocol becomes permisionless it might be a big problem
## Recommendation
Validate admin before trusting them properly

# [L-03] Do not use msg.sender as delegatee in connext.xcall
Although the functions are restricted to admin related, the issue is mostly mitigated, but it's better not to use delegatee as msg.sender
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol#L434
## Recommendation
Use the admin configurable address in this cases
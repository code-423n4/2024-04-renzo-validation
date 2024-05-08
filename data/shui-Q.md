# [L-01] Bad oracle price will DoS the system
Both the minting and deposit use the calculateTVL function that internally calculates each asset's value using chainlink. If the oracle mal functions the for loop will revert and the user will not be able to mint or deposit. This is a high severity issue as it can cause a DoS attack on the system.

## Context
`calculateTVL()`
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L316-L321
`withdraw()`
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L217-L232
`deposit()`
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L504-L575

## Recommendation
Consider implementing a fallback mechanism in case the oracle fails. This can be done by using a different oracle (Uniswap) or by using a different mechanism to calculate the TVL. This will ensure that the system does not get stuck in case of an oracle failure.

# [L-02] Use of transfer instead of call{value: x}() in the xRenzoBridge contract
The xRenzoBridge contract uses the transfer function to send funds to a determined address. This is not recommended as the transfer function only provides 2300 gas for its operation. This means that the recipient must have a payable callback and the callback must not use more than 2300 gas. If the recipient does not have a payable callback or the callback uses more than 2300 gas, the transfer will fail. This can lead to the locking of funds and the user will not be able to receive the funds.

## Context
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L294-L296

## Recommendation
Use the call{value: x}() function instead of transfer to send funds to a determined address. This will ensure that the recipient receives the funds even if they do not have a payable callback or the callback uses more than 2300 gas.

# [L-03] Any one call fordward the ETH in the RewardHandler

The `receive` function in the RewardHandler contract can be called by anyone and forward the ETH. Notice that it does not have any event being emitted, so it is not possible to track who is sending the ETH to the contract.

## Context
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Rewards/RewardHandler.sol#L52-L70

## Recommendation
Consider adding an event to the `_forwardETH` function to track who is sending the ETH to the contract. This will ensure that the contract owner can track who is sending the ETH to the contract.
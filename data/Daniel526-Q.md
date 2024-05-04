## A. Reentrancy Vulnerability in `_withdraw` Function
[XERC20Lockbox.sol#L125-L136](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L125-L136)
In the `_withdraw` function, the contract emits a `Withdraw` event, burns the specified amount of XERC20 tokens from the caller's balance, and then interacts with external contracts to transfer tokens or native assets. However, the interaction step occurs before the effect step, which violates the Check-Effect-Interact (CEI) pattern, potentially leading to a reentrancy vulnerability.
### Impact:
This vulnerability could enable malicious actors to exploit reentrancy attacks. By calling back into the contract before the effect step (burning tokens) is completed, attackers could manipulate the contract's state, leading to unexpected behavior or loss of funds.

### Mitigation:

To address this vulnerability, we need to ensure that interactions with external contracts occur after completing the effect step (burning tokens). This sequence helps prevent reentrancy attacks by updating the contract's state before interacting with external entities.
# Some contracts inheriting `ReentrancyGuardUpgradeable` are missing a call to `__ReentrancyGuard_init`.

The following three contracts inherit from `ReentrancyGuardUpgradeable`, but they do not call `__ReentrancyGuard_init` in the `initialize` function like other contracts.

- https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L16-L21
- https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L27-L32
- https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L11-L15

# Notice mistakes and mismatches: [IRoleManager.sol#49](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Permissions/IRoleManager.sol#L49C8-L49C9), [IRoleManager.sol#17](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Permissions/IRoleManager.sol#L17C1-L17C111)

## Notice mistakes
[IRoleManager.sol#49](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Permissions/IRoleManager.sol#L49C8-L49C9)

```solidity
// @audit haas ---> has
49: -    /// @dev Determine if the specified address haas permission to update Withdraw Queue params
    +    /// @dev Determine if the specified address has permission to update Withdraw Queue params
```

[IRoleManager.sol#17](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Permissions/IRoleManager.sol#L17C1-L17C111)
```solidity
// @audit double config
17: -    /// @dev Determines if the specified address has permission to update config on the Oracle Contract config
    +   /// @dev Determines if the specified address has permission to update config on the Oracle Contract
```
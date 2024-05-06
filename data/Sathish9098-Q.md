##

## [L-1] Misguided Zero-Value Checks for uint256

Since ``uint256`` cannot be negative, the check ``_amount < 0`` is inherently impossible. This leaves only the possibility of _amount being equal to zero as a valid check.

```solidity
FILE: 2024-04-renzo/contracts/Bridge/Connext/integratio/LockboxAdapterBlast.sol

65: if (_amount <= 0) {

```
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L65

### Recommended Mitigation
```solidity

if (_amount == 0) {

```






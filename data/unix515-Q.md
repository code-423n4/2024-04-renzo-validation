### These issues are not listed in the automated findings report.

# 1. No zero check for input parameters on RestakeManager#`initialize()`

### Description

RestakeManager#`initialize()` has no validation check for input parameters.

### Impact

It might happens to take ownership of the contract, and in the best case forcing a re-deployment

### Links to affected code
 
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L101-L118
### Recommended Mitigation Steps

```diff
function initialize(
	IRoleManager _roleManager,
	IEzEthToken _ezETH,
	IRenzoOracle _renzoOracle,
	IStrategyManager _strategyManager,
	IDelegationManager _delegationManager,
	IDepositQueue _depositQueue
) public initializer {

	__ReentrancyGuard_init();
++	if (
++		address(_roleManager) == address(0) ||
++		address(_ezETH) == address(0) ||
++		address(_renzoOracle) == address(0) ||
++		address(_strategyManager) == address(0) ||
++		address(_delegationManager) == address(0) ||
++		address(_depositQueue) == address(0)
++	) {
++		revert InvalidZeroInput();
++	}
        
	roleManager = _roleManager;
	ezETH = _ezETH;
	renzoOracle = _renzoOracle;
	strategyManager = _strategyManager;
	delegationManager = _delegationManager;
	depositQueue = _depositQueue;
	paused = false;
}
```

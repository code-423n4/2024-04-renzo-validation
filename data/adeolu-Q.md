 # [L-01] - EzETHToken initialized with wrong name and symbol 

```
    function initialize(IRoleManager _roleManager) public initializer {
        if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();

        __ERC20_init("ezETH", "Renzo Restaked ETH"); //@audit: name and symbol are mixed up. token is intitalized wrongly. symbol is used as name and name is used as symbol 
        roleManager = _roleManager;
    }
```

the `__ERC20_init()` function has the `name` and `symbol` parameters as seen [here](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/683b6729cd061ceecf0ebdc29b00a01930fcc8e7/contracts/token/ERC20/ERC20Upgradeable.sol#L58). but in the code, ` ezETH` which is the token symbol is placed as input for parameter `name_` and `Renzo Restaked ETH` which is token name is used as input for the parameter `symbol_`. 


## Recommended Mitigation
use `Renzo Restaked ETH` ezETH as input for parameter `name_` and use `ezETH` as input for parameter `symbol_` 
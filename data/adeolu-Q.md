 # [L-01] - EzETHToken initialized with wrong name and symbol 

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/token/EzEthToken.sol#L36

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


# [L-02] - Can enforce eth direct transfer to be from only WETH contract and revert for users. 

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L134C1-L137C80

```
     * @notice  WARNING: This function does NOT whitelist who can send funds from the L2 via Connext.  Users should NOT
     *          send funds directly to this contract.  A user who sends funds directly to this contract will cause
     *          the tokens on the L2 to become over collateralized and will be a "donation" to protocol.  Only use
     *          the deposit contracts on the L2 to send funds to this contract.
    function xReceive(
```

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L309C1-L313C34
```
    /**
     * @notice Fallback function to handle ETH sent to the contract from unwrapping WETH
     * @dev Warning: users should not send ETH directly to this contract!
     */
    receive() external payable {}
```

The warnings above says that "users should not send ETH directly to this contract!" else the eth sent will become a "donation" to the protocol and the fallback should only be used for unwrapping WETH. But still to prevent/totally eliminate user error, mistakes etc a strict check can be put in the fallback logic to reject eth transfers made by addresses that are not the registered WETH contract. Something like the logic below should do. 

```
    receive() external payable {
        //@audit: to enforce the warning you can do something like
        if (msg.sender != address(wETH)) revert();
    }
```

This will eliminate all posibilities of users making any erroreous use of the contract and "donating" to the protocol like the warnings above specifies. 

## Recommended Mitigation
modify the fallback to be like below
```
    receive() external payable {
        //@audit: to enforce the warning you can do something like
        if (msg.sender != address(wETH)) revert();
    }
```
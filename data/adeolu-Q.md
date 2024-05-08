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


# [L-03] -  xRenzoBridge ConnextMessageSent events are not distinct, they have no distinct identifier

messages sent via connext's xcall() function all have a `transferID` value which is returned on sucessful execution of a cross chain message send request. This `transferID` is unique for every message sent via connext. The xRenzoBridge event `ConnextMessageSent` is emitted on every sucessful message request via connext but it's parameters are not unique enough. 

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L275C1-L280C15

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L43C1-L48C7
```
    event ConnextMessageSent(
        uint32 indexed destinationChainDomain, // The chain domain Id of the destination chain.
        address receiver, // The address of the receiver on the destination chain.
        uint256 exchangeRate, // The exchange rate sent.
        uint256 fees // The fees paid for sending the Connext message.
    );
```

destinationChainDomain, receiver, exchangeRate, fees can be the same for two events. i.e if two transactions are made at the same time, bot events can have same value for timestamp,  destinationChainDomain, receiver, exchangeRate and fees. 

It is better to inclued the `transferID` value which is returned by the [xcall()](https://github.com/connext/monorepo/blob/8338d6506c609f9383d81133c3cb40cfb9e44392/packages/deployments/contracts/contracts/core/connext/facets/BridgeFacet.sol#L299C1-L307C59) fcn in the event parameters as well. This will make each event unique to each connext message and allow for easier identification/relationship between each message and their `ConnextMessageSent` events.

## Recommended Mitigation
modify the `ConnextMessageSent` event to be like below

```
    event ConnextMessageSent(
   bytes32 transferID //id of the connext cross chain message. 
        uint32 indexed destinationChainDomain, // The chain domain Id of the destination chain.
        address receiver, // The address of the receiver on the destination chain.
        uint256 exchangeRate, // The exchange rate sent.
        uint256 fees // The fees paid for sending the Connext message.
    );
```

# [L-04] -  OperatorDelegator.setTokenStrategy()  should check that each strategy address being set is whitelisted for deposit in eigenlayer strategy manager

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol#L106C1-L115C1
```
    /// @dev Sets the strategy for a given token - setting strategy to 0x0 removes the ability to deposit and withdraw token
    function setTokenStrategy(
        IERC20 _token,
        IStrategy _strategy
    ) external nonReentrant onlyOperatorDelegatorAdmin {
        if (address(_token) == address(0x0)) revert InvalidZeroInput();

        tokenStrategyMapping[_token] = _strategy; `strategyIsWhitelistedForDeposit` mapping first.
        emit TokenStrategyUpdated(_token, _strategy);
    }
```
`setTokenStrategy()` updateds the `tokenStrategyMapping` for each token to a strategy but it doesnt confirm that the strategy is whitelisted for deposits by the eigenlayer strategy manager. In cases where an unwhitelisted strategy is updated into storage the `deposit()` function will fail because             [strategyManager.depositIntoStrategy(tokenStrategyMapping[_token], _token, _tokenAmount)](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol#L168) will not recognise the strategy address as it is not whitelisted for deposit. 

`setTokenStrategy()` should confirm if a strategy is whitelisted for deposit by calling the eigenlayer public mapping [strategyIsWhitelistedForDeposit](https://github.com/Layr-Labs/eigenlayer-contracts/blob/ef2ea4a7459884f381057aa9bbcd29c7148cfb63/src/contracts/core/StrategyManagerStorage.sol#L60) and checking that it indeed returns true. 

## Recommended Mitigation
modify the `setTokenStrategy()`fcn to be like below

```
    function setTokenStrategy(
        IERC20 _token,
        IStrategy _strategy
    ) external nonReentrant onlyOperatorDelegatorAdmin {
        if (address(_token) == address(0x0)) revert InvalidZeroInput();
        if(strategyManager.strategyIsWhitelistedForDeposit(_strategy) != true) revert stategyNotWhitelisted()

        tokenStrategyMapping[_token] = _strategy; //@audit this mapping can be updated with a straategy that is not whitelisted for deposit in eigenlayer. it should check if the straegy is whitelisted by calling `strategyIsWhitelistedForDeposit` mapping first.
        emit TokenStrategyUpdated(_token, _strategy);
    }
```
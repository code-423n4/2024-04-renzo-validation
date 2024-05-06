# 1. Previous stategy deposits not migrated when new asset strategy is set

Links to affected code *

https://github.com/code-423n4/2024-04-renzo/blob/1c7cc4e632564349b204b4b5e5f494c9b0bc631d/contracts/Delegation/OperatorDelegator.sol#L106

https://github.com/code-423n4/2024-04-renzo/blob/1c7cc4e632564349b204b4b5e5f494c9b0bc631d/contracts/Delegation/OperatorDelegator.sol#L327

## Impact

If a new strategy is set, there's no check to see if there's an existing `strategy` or transfer the current's strategy balance to the new one. Setting a new stategy doesn't migrate previous stategy's tokens to the new one which and can cause previous balances to be overwritten and issues with accounting.

```solidity
    function setTokenStrategy(
        IERC20 _token,
        IStrategy _strategy
    ) external nonReentrant onlyOperatorDelegatorAdmin {
        if (address(_token) == address(0x0)) revert InvalidZeroInput();

        tokenStrategyMapping[_token] = _strategy;
        emit TokenStrategyUpdated(_token, _strategy);
    }
```
The token balance is gotten from the `getTokenBalanceFromStrategy` function which will be overwritten if a new asset strategy is set. The value is used in calculating TVLs which is used extensively in assets deposits and redemptions.
```solidity
    function getTokenBalanceFromStrategy(IERC20 token) external view returns (uint256) { 
        return
            queuedShares[address(this)] == 0
                ? tokenStrategyMapping[token].userUnderlyingView(address(this))
                : tokenStrategyMapping[token].userUnderlyingView(address(this)) +
                    tokenStrategyMapping[token].sharesToUnderlyingView(
                        queuedShares[address(token)]
                    );
    }
```
***

# 2. Should introduce a minimum `ezETH` amount parameter to receive upon deposits.

Lines of code* 

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L491

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L592

### Impact

When users deposit into the protocol through the `deposit` and `depositETH` function, there's no way for intoduce the minimum amount of ezETH they'd like to receive. Considering that the calculated ezETH is dependent on oracle prices of the collateral token being deposited, the value of which is used to calculate how much ezETH to mint to the users, they have no way of predicting how many `ezETH` they will get back at the moment of minting, as the prices could be updated while the request is in the mempool. This leaves them vulnerable to unfavourable slippage or sandwich attacks.

```solidity
    function deposit(
        IERC20 _collateralToken,
        uint256 _amount,
        uint256 _referralId
    ) public nonReentrant notPaused {
        // Verify collateral token is in the list - call will revert if not found
        uint256 tokenIndex = getCollateralTokenIndex(_collateralToken);

        // Get the TVLs for each operator delegator and the total TVL
        (
            uint256[][] memory operatorDelegatorTokenTVLs,
            uint256[] memory operatorDelegatorTVLs,
            uint256 totalTVL
        ) = calculateTVLs();

        // Get the value of the collateral token being deposited
        uint256 collateralTokenValue = renzoOracle.lookupTokenValue(_collateralToken, _amount);
...
        // Calculate how much ezETH to mint
        uint256 ezETHToMint = renzoOracle.calculateMintAmount(
            totalTVL,
            collateralTokenValue,
            ezETH.totalSupply()
        );

        // Mint the ezETH
        ezETH.mint(msg.sender, ezETHToMint);

        // Emit the deposit event
        emit Deposit(msg.sender, _collateralToken, _amount, ezETHToMint, _referralId);
    }
```
### Recommended Mitigation Steps

Introduce a minimum amount out parameter.

***

# 3. `setRenzoDeposit` should check if contract is paused

Lines of code*

https://github.com/code-423n4/2024-04-renzo/blob/1c7cc4e632564349b204b4b5e5f494c9b0bc631d/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L130

### Impact

From the comment in the constructor of ConnextReceiver.sol, the contract is paused so as to be able to setup `xRenzoDeposit` address 

```solidity
    constructor(address _connext, address _xRenzoBridgeL1, uint32 _connextEthChainDomain) {
...
        // Pause The contract to setup xRenzoDeposit
        _pause();
    }
```

And the `setRenzoDeposit` is used to set or potentially update the `xRenzoDeposit` address, but it's missing a check for contract's current paused status, meaning that the address can be set up whether the contract is paused or not, which goes against the desired behaviour of the contract. 

```solidity
    /**
     * @notice This function updates the xRenzoDeposit Contract address
     * @dev This should be a permissioned call (onlyOnwer)
     * @param _newXRenzoDeposit New xRenzoDeposit Contract address
     */
    function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {
        if (_newXRenzoDeposit == address(0)) revert InvalidZeroInput();
        emit XRenzoDepositUpdated(_newXRenzoDeposit, address(xRenzoDeposit));
        xRenzoDeposit = IxRenzoDeposit(_newXRenzoDeposit);
    }
```

### Recommended Mitigation Steps
Consider introducing a `whenPaused` modifier or check if contract is paused before updating the address.

***



# 4. Unspend allowances can break contract, should approve to 0 first

Lines of code* 

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L84-L86

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L181

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L369

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L429

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L164

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L297

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L142

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L552

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L559

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L664

### Impact
None of the various use of the `safeApprove` function approved to zero. Openzeppelin's `safeApprove` function is used to prevent approval front-runs by enforcing first approving to 0, then to a new amount. It's implemented as following:

```solidity
    function safeApprove(
        IERC20 token,
        address spender,
        uint256 value
    ) internal {
        // safeApprove should only be called when setting an initial allowance,
        // or when resetting it to zero. To increase and decrease it, use
        // 'safeIncreaseAllowance' and 'safeDecreaseAllowance'
        require(
            (value == 0) || (token.allowance(address(this), spender) == 0),
            "SafeERC20: approve from non-zero to non-zero allowance"
        );
        _callOptionalReturn(token, abi.encodeWithSelector(token.approve.selector, spender, value));
    }
```
For instance, in the `_trade` function in xRenzoDeposit.sol, in a case of an very extremly profitable swap in which not all approved tokens are used, the remaining unspent allowance, not zeroed out will cause any subsequent calls to approve connext to fail.

```solidity
    function _trade(uint256 _amountIn, uint256 _deadline) internal returns (uint256) {
        // Approve the deposit asset to the connext contract
        depositToken.safeApprove(address(connext), _amountIn);
...
        // Swap the tokens
        uint256 amountNextWETH = connext.swapExact(
            swapKey,
            _amountIn,
            address(depositToken),
            address(collateralToken),
            minOut,
            _deadline
        );

...
    }
```

### Recommended Mitigation Steps

Consider zeroing out approvals after transaction is done.

***

# 5. `getMintRate` reverting when there's no receiver and no oracle makes price updates by owner redundant.

Lines of code* 

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol#L320

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol#L291

### Impact
The contract owner can update token price. This can be done, regardless of if there's an oracle or a receiver or not.

```solidity
    function updatePriceByOwner(uint256 price) external onlyOwner {
        return _updatePrice(price, block.timestamp);
    }
```

The issue is that this fact is not taken into consideration when getting mint rate. The function reverts if there's no recevier and no oracle, tentatively ignoring the price updates from the owner. 

```solidity
    function getMintRate() public view returns (uint256, uint256) {
        // revert if PriceFeedNotAvailable
        if (receiver == address(0) && address(oracle) == address(0)) revert PriceFeedNotAvailable();
        if (address(oracle) != address(0)) {
            (uint256 oraclePrice, uint256 oracleTimestamp) = oracle.getMintRate();
            return
                oracleTimestamp > lastPriceTimestamp
                    ? (oraclePrice, oracleTimestamp)
                    : (lastPrice, lastPriceTimestamp);
        } else {
            return (lastPrice, lastPriceTimestamp);
        }
    }
```
Even if owner updates the prices constantly, as long as both receivers and oracles are down, the price updates will be potentially useless, rendering the function more or less redundant.

### Recommended Mitigation Steps
Consider removing the check, if price feed has not been updated by the owner in a while, the `lastPriceTimestamp` will revert in the `deposit` function where it's used as it reverts when timestamp is stale. This prevents the use of stale prices while also making use of the prices as updated by the owner.

```solidity
    function _deposit(
        uint256 _amountIn,
        uint256 _minOut,
        uint256 _deadline
    ) internal returns (uint256) {
...
        // Fetch price and timestamp of ezETH from the configured price feed
        (uint256 _lastPrice, uint256 _lastPriceTimestamp) = getMintRate();

        // Verify the price is not stale
        if (block.timestamp > _lastPriceTimestamp + 1 days) {
            revert OraclePriceExpired();
        }
```

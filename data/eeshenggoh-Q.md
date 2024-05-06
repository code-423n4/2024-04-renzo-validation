Word typo in `xRenzoBridge.sol`, should re-write from `xRenzoDepsot` into `xRenzoDeposit`:
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L220
 
Word typo in `xRenzoDeposit.sol`, should re-write from `depoist` into `deposit`:

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol#L125

============================================================

Extra codes are written but not removed, instead it was commented out. Wasting space and readability.
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol#L43

============================================================

Contract implementation is not consistent in the way it is written.
For example in `OperatorDelegator.sol`:
```solidity
   /// @dev Allows only the RestakeManager address to call functions
    // @audit-issue low, not using same consistency for access control
    modifier onlyRestakeManager() {
        if (msg.sender != address(restakeManager)) revert NotRestakeManager();
        _;
    }
```

There is missing consistency in how the code was written by developers. Most contracts uses the pause library from Openzeppelin, however in `RestakeManager.sol` the code is written without using the library.
```solidity
    function initialize(
    -- SNIP -- 
    ) public initializer {
        __ReentrancyGuard_init();

        roleManager = _roleManager;
        ezETH = _ezETH;
        renzoOracle = _renzoOracle;
        strategyManager = _strategyManager;
        delegationManager = _delegationManager;
        depositQueue = _depositQueue;
        paused = false; // @audit-issue (low) should use openzeppelin pausable for consistency
    }
    /// @dev Only allows execution if contract is not paused
    modifier notPaused() {
        if (paused) revert ContractPaused();
        _;
    }
    function setPaused(bool _paused) external onlyDepositWithdrawPauserAdmin {
        paused = _paused;
    }
```
============================================================
These storage state variables are not used and depreciated should be removed from codebase:
```solidity
    /// @dev data stored for a withdrawal
    //@audit-issue  not implemented should remove
    struct PendingWithdrawal {
        uint256 ezETHToBurn;
        address withdrawer;
        IERC20 tokenToWithdraw;
        uint256 tokenAmountToWithdraw;
        IOperatorDelegator operatorDelegator;
        bool completed;
    }

    /// @dev mapping of pending withdrawals, indexed by the withdrawal root from EigenLayer
    //@audit-issue  not implemented
    mapping(bytes32 => PendingWithdrawal) public pendingWithdrawals;
```
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManagerStorage.sol#L29
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManagerStorage.sol#L39
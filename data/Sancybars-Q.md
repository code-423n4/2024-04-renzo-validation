wrong natspec can confuse users about which role is meant to pause deposit  in `RestakeManager` contract 

`RestakeManager` is the main entry point into the protocol for external users, users deposit into this contract to interact with the EigenLayer. However this function only be paused by the `DepositWithdrawPauserAdmin` role but the code comments states otherwise which might confuse users 
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts%2FRestakeManager.sol#L120-L122
```solidity
    /// @dev Allows a restake manager admin to set the paused state of the contract
    function setPaused(bool _paused) external onlyDepositWithdrawPauserAdmin {
        paused = _paused;
    }
```
According to the comment the `RestakeManagerAdmin` role is meant to call this function to pause the deposit into the contract users will think the `DepositWithdrawPauserAdmin` role have unauthorized access to the function while the role is the rightful role to call the function to pause deposits.
## fix 
Correct the code comment to define the role able to call the function as Deposit withdraw pauser admin.

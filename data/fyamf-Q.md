# Q1
## Impact
When certain functions like `addOperatorDelegator`, `removeOperatorDelegator`, or `setOperatorDelegatorAllocation` are used, there's a problem. It doesn't properly check if the shares given to operator delegators are fair. This could mean some delegators get more than they should, which isn't right.

## Proof of Concept
In the code, when you use `setOperatorDelegatorAllocation` or `addOperatorDelegator`, it only checks if the share given is more than 100%. But it doesn't check if all shares together add up to exactly 100%.
```solidity
if (_allocationBasisPoints > (100 * BASIS_POINTS)) revert OverMaxBasisPoints();
```
https://github.com/code-423n4/2024-04-renzo/blob/1c7cc4e632564349b204b4b5e5f494c9b0bc631d/contracts/RestakeManager.sol#L192
https://github.com/code-423n4/2024-04-renzo/blob/1c7cc4e632564349b204b4b5e5f494c9b0bc631d/contracts/RestakeManager.sol#L146

For example, if you have three delegators: OD1, OD2, OD3, and you assign them 50%, 50%, 50% respectively, it will give all the deposits to OD1 and OD2. OD3 will get nothing. This happens because when the function `deposit` is called, it checks which delegator is not meeting its threshold to be selected for the deposit destination. Since OD1 and OD2 should take %100 of the TVL, nothing will be left for OD3.
```solidity
// Determine which operator delegator to use
IOperatorDelegator operatorDelegator = chooseOperatorDelegatorForDeposit(
    operatorDelegatorTVLs,
    totalTVL
);
```
https://github.com/code-423n4/2024-04-renzo/blob/1c7cc4e632564349b204b4b5e5f494c9b0bc631d/contracts/RestakeManager.sol#L533
```solidity
        for (uint256 i = 0; i < tvlLength; ) {
            if (
                tvls[i] <
                (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
                    BASIS_POINTS /
                    BASIS_POINTS
            ) {
                return operatorDelegators[i];
            }

            unchecked {
                ++i;
            }
        }
```
https://github.com/code-423n4/2024-04-renzo/blob/1c7cc4e632564349b204b4b5e5f494c9b0bc631d/contracts/RestakeManager.sol#L376

Moreover, when the function `removeOperatorDelegator` is called, it does not update the allocation of active operator delegators.

## Tools Used

## Recommended Mitigation Steps
To fix this, whenever you change the shares or add/remove a delegator, you should update the shares properly. You can do this by using a function like the one below:

```solidity
function updateAllocations() public {
    uint256 allocationsSum;
    // Calculate the total sum of allocations
    for (uint256 i = 0; i < operatorDelegators.length; ++i) {
        allocationsSum += operatorDelegatorAllocations[operatorDelegators[i]];
    }
    // Update each delegator's allocation proportionally
    for (uint256 i = 0; i < operatorDelegators.length; ++i) {
        operatorDelegatorAllocations[operatorDelegators[i]] =
            (operatorDelegatorAllocations[operatorDelegators[i]] * BASIS_POINTS) /
            allocationsSum;
    }
}
```
This function makes sure each delegator gets a fair share based on the total sum of allocations.


# Q2

## Impact
The role assigned to pause withdrawals is incorrect.

## Proof of Concept
The protocol documentation specifies that only the role `DEPOSIT_WITHDRAW_PAUSER` should be allowed to pause/unpause deposits and withdrawals. However, the actual implementation allows only the role `WITHDRAW_QUEUE_ADMIN` to pause withdrawals, not `DEPOSIT_WITHDRAW_PAUSER`.
```solidity
    /**
     * @notice  Pause the contract
     * @dev     Permissioned call (onlyWithdrawQueueAdmin)
     */
    function pause() external onlyWithdrawQueueAdmin {
        _pause();
    }

    /**
     * @notice  Unpause the contract
     * @dev     Permissioned call (onlyWithdrawQueueAdmin)
     */
    function unpause() external onlyWithdrawQueueAdmin {
        _unpause();
    }
```
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L135-L149

```solidity
    /// @dev Allows only Withdraw Queue Admin to configure the contract
    modifier onlyWithdrawQueueAdmin() {
        if (!roleManager.isWithdrawQueueAdmin(msg.sender)) revert NotWithdrawQueueAdmin();
        _;
    }
```
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L38-L42

```soldity
    /// @dev Determine if the specified address haas permission to update Withdraw Queue params
    /// @param potentialAddress Address to check
    function isWithdrawQueueAdmin(address potentialAddress) external view returns (bool) {
        return hasRole(WITHDRAW_QUEUE_ADMIN, potentialAddress);
    }
```
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L96-L100




### Simplified Mitigation Steps
To align the implementation with the documentation, modify the modifier of the `pause()` and `unpause()` functions to `onlyDepositWithdrawPauserAdmin`.
```solidity
/**
 * @notice Pause the contract
 * @dev Permissioned call (onlyDepositWithdrawPauserAdmin)
 */
function pause() external onlyDepositWithdrawPauserAdmin {
    _pause();
}

/**
 * @notice Unpause the contract
 * @dev Permissioned call (onlyDepositWithdrawPauserAdmin)
 */
function unpause() external onlyDepositWithdrawPauserAdmin {
    _unpause();
}
```

# Q3
## Impact
The pausing mechanism defined in the WithdrawQueue contract is not utilized.

## Proof of Concept
The WithdrawQueue contract contains functions `pause()` and `unpause()` to pause and unpause the contract, respectively. However, these functions are not called anywhere within the protocol. In critical situations such as encountering bugs or potential attacks, where pausing the protocol is necessary, this mechanism is not utilized, leading to potential risks.
```solidity
    /**
     * @notice  Pause the contract
     * @dev     Permissioned call (onlyWithdrawQueueAdmin)
     */
    function pause() external onlyWithdrawQueueAdmin {
        _pause();
    }

    /**
     * @notice  Unpause the contract
     * @dev     Permissioned call (onlyWithdrawQueueAdmin)
     */
    function unpause() external onlyWithdrawQueueAdmin {
        _unpause();
    }
```
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L135-L149



## Tools Used

## Recommended Mitigation Steps
Define a modifier `notPaused` and apply it to critical functions such as `withdraw` and `claim` to ensure that the protocol cannot be interacted with while paused:
```solidity
modifier notPaused() {
    if (paused) revert ContractPaused();
    _;
}
```

# Q4
## Impact
There is a conflict between roles in setting the cooldown period to a large value.

## Proof of Concept
According to the protocol documentation, only the role `DEPOSIT_WITHDRAW_PAUSER` should be authorized to pause/unpause withdrawals. However, if the role `WITHDRAW_QUEUE_ADMIN` sets the `coolDownPeriod` to the maximum possible value (`type(uint256).max`), it effectively pauses all withdrawals.
```solidity
    /**
     * @notice Updates the coolDownPeriod for withdrawal requests
     * @dev    It is a permissioned call (onlyWithdrawQueueAdmin)
     * @param   _newCoolDownPeriod  new coolDownPeriod in seconds
     */
    function updateCoolDownPeriod(uint256 _newCoolDownPeriod) external onlyWithdrawQueueAdmin {
        if (_newCoolDownPeriod == 0) revert InvalidZeroInput();
        emit CoolDownPeriodUpdated(coolDownPeriod, _newCoolDownPeriod);
        coolDownPeriod = _newCoolDownPeriod;
    }
```
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L124-L133



## Tools Used

## Recommended Mitigation Steps
Update the `updateCoolDownPeriod()` function to include a maximum acceptable value for the cooldown period:
```solidity
function updateCoolDownPeriod(uint256 _newCoolDownPeriod) external onlyWithdrawQueueAdmin {
    if (_newCoolDownPeriod == 0) revert InvalidZeroInput();
    require(_newCoolDownPeriod <= 7*24*60*60, "maximum acceptable value is one week");
    emit CoolDownPeriodUpdated(coolDownPeriod, _newCoolDownPeriod);
    coolDownPeriod = _newCoolDownPeriod;
}
```
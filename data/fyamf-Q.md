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
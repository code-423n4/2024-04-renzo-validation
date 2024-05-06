- L-01
## Description
Zero address operatorDelegator can be added and given allocation too.

## Details
In the `RestakeManager.sol::addOperatorDelegator`, a zero address operatorDelegator can be added and later in the same function, it is assigned allocation too. This zero address check is not existing in this function but it is there in other functions such as the `setOperatorDelegatorAllocation`.

```solidity
// @audit no zero address check for _newOperatorDelegator
function addOperatorDelegator(
        IOperatorDelegator _newOperatorDelegator,
        uint256 _allocationBasisPoints
    ) external onlyRestakeManagerAdmin {
        // Ensure it is not already in the list
        uint256 odLength = operatorDelegators.length;
        for (uint256 i = 0; i < odLength; ) {
            if (address(operatorDelegators[i]) == address(_newOperatorDelegator))
                revert AlreadyAdded();
            unchecked {
                ++i;
            }
        }

        // Verify a valid allocation
        if (_allocationBasisPoints > (100 * BASIS_POINTS)) revert OverMaxBasisPoints();

        // Add it to the list
        operatorDelegators.push(_newOperatorDelegator);

        emit OperatorDelegatorAdded(_newOperatorDelegator);

        // Set the allocation
        operatorDelegatorAllocations[_newOperatorDelegator] = _allocationBasisPoints;

        emit OperatorDelegatorAllocationUpdated(_newOperatorDelegator, _allocationBasisPoints);
    }
```
## Code Reference
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L131-L157

- L-02
## Description
No zero address check in `xRenzoBridge.sol::recoverNative` can lead to permanent loss of Native Token.
## Details
In the `xRenzoBridge.sol::recoverNative`, there is no zero address check.
```solidity
    // @audit no zero address checks
    function recoverNative(uint256 _amount, address _to) external onlyBridgeAdmin {
        payable(_to).transfer(_amount);
    }
```
This can lead to the funds being permanently lost.
## Code Reference
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol#L478-L480
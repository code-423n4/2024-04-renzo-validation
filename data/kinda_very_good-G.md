[G1] RestakeManager:BASIS_POINTS  can be initialized to 10000 to save gas instead of multiplying by 100 when ever it is to be used 

[G2] cache storage variable that are used multiple times are in loops in memory instead of calling sload multiple times 

instances 
1
```

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

The entire operatorDelegators array could be copied to memory and looped through 
```
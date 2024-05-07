### Report 1:
#### Setting Current Limit to Zero would break Protocol
The code provided below from the XERC20 contract shows how New current limit is calculated, Under certain circumstances as noted from the pointer when _currentLimit  is less than oldmaxlimit to newmaxlimit difference a value of zero is assigned to _newCurrentLimit, this would cut short the proper range in limit difference which would break protocol, Protocol should either revert under this circumstance or work with other mitigations to stop this.
https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L285
```solidity
 function _calculateNewCurrentLimit(
        uint256 _limit,
        uint256 _oldLimit,
        uint256 _currentLimit
    ) internal pure returns (uint256 _newCurrentLimit) {
        uint256 _difference;

        if (_oldLimit > _limit) {
            _difference = _oldLimit - _limit;
>>>            _newCurrentLimit = _currentLimit > _difference ? _currentLimit - _difference : 0;
        } else {
            _difference = _limit - _oldLimit;
            _newCurrentLimit = _currentLimit + _difference;
        }
    }
```


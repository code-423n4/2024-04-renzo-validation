### QA-1 Evaluate whether all gas accounting and gas returns are truly necessary. Consider checking again

File: OperatorDelegator.sol

Impact: 
```solidity
    function _recordGas(uint256 initialGas) internal {
        uint256 gasSpent = (initialGas - gasleft() + baseGasAmountSpent) * tx.gasprice;
        adminGasSpentInWei[msg.sender] += gasSpent;
        emit GasSpent(msg.sender, gasSpent);
    }
```

Recommended:
Take to reduce or eliminate unimportant gas logic
```
    if (adminGasSpentInWei[msg.sender] > 0) {
```

### QA-2 Lack of Storage Gap in Upgradeable Contracts 

File: OperatorDelegator.sol , RestakeManager.sol , xRenzoBridge.sol, xRenzoDeposit.sol
For upgradeable contracts, there must be storage gap to “allow developers to freely add new state variables in the future without compromising the storage compatibility with existing deployments” (quote OpenZeppelin). Otherwise it may be very difficult to write new implementation code. Without storage gap, the variable in child contract might be overwritten by the upgraded base contract if new variables are added to the base contract. This could have unintended and very serious consequences to the child contracts, potentially causing loss of user fund or cause the contract to malfunction completely.

Refer to the bottom part of this article: https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable

Recommended: 
Recommend adding appropriate storage gap at the end of upgradeable contracts such as the below. Please reference OpenZeppelin upgradeable contract templates.

### QA-3 In the sendPrice, the payable keyword is not necessary. Remove it to avoid unnecessary gas consumption.
File:xRenzoBridge.sol
```solidity
  function sendPrice(
        CCIPDestinationParam[] calldata _destinationParam,
        ConnextDestinationParam[] calldata _connextDestinationParam
    ) external payable onlyPriceFeedSender nonReentrant {
 ```
 Recommended: 
 Consider Remove payable
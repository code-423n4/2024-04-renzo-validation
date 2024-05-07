##

## [L-1] Misguided Zero-Value Checks for uint256

Since ``uint256`` cannot be negative, the check ``_amount < 0`` is inherently impossible. This leaves only the possibility of _amount being equal to zero as a valid check.

```solidity
FILE: 2024-04-renzo/contracts/Bridge/Connext/integratio/LockboxAdapterBlast.sol

65: if (_amount <= 0) {

```
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L65

### Recommended Mitigation
```solidity

if (_amount == 0) {

```

##

## [L-] Inconsistency in Timestamp Verification Across ``RenzoOracleL2`` and ``xRenzoDeposit`` Contracts

There is contradiction in timestamp check in 2 places. One place checks 24 hours + 1 minute and another place check exactly 24 hours. 

```solidity
FILE: 2024-04-renzo/contracts/Bridge/L2/Oracle
/RenzoOracleL2.sol

13:  uint256 public constant MAX_TIME_WINDOW = 86400 + 60; // 24 hours + 60 seconds

52: if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();

```
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L52

```solidity 
FILE: 2024-04-renzo/contracts/Bridge/L2/xRenzoDeposit.sol

52: if (block.timestamp > _lastPriceTimestamp + 1 days) {

```
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol#L248

##

## [L-3] Risk of Unchecked Oracle Price Anomalies in ``lookupTokenValue()`` function

Oracles serve as bridges between external data sources and the blockchain. However, the integrity of the data they provide can be compromised by errors in data feeds, manipulation by attackers, or even technical malfunctions.

### Impact

- Sudden, unrealistic spikes or drops in price data can occur, which do not accurately reflect market conditions.

- If an attacker can influence the oracle, either directly by tampering with the data source or indirectly through market manipulation, they might feed incorrect prices to the smart contract.

```solidity
FILE: 2024-04-renzo/contracts/Oracle/RenzoOracle.sol

 /// @dev Given a single token and balance, return value of the asset in underlying currency
    /// The value returned will be denominated in the decimal precision of the lookup oracle
    /// (e.g. a value of 100 would return as 100 * 10^18)
    function lookupTokenValue(IERC20 _token, uint256 _balance) public view returns (uint256) {
        AggregatorV3Interface oracle = tokenOracleLookup[_token];
        if (address(oracle) == address(0x0)) revert OracleNotFound();

        (, int256 price, , uint256 timestamp, ) = oracle.latestRoundData();
        if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();
        if (price <= 0) revert InvalidOraclePrice();

        // Price is times 10**18 ensure value amount is scaled
        return (uint256(price) * _balance) / SCALE_FACTOR;
    }

```
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol#L68-L81

##

## [L-] Risk of Suboptimal Delegator Selection Due to First-Match Approach in ``chooseOperatorDelegatorForDeposit()`` 

Multiple delegators have TVLs (Total Value Locked) below their respective thresholds, the function chooseOperatorDelegatorForDeposit will select and return the first delegator that meets this condition as it iterates through the list of delegators. This means that the function does not compare all delegators to find the one with the lowest TVL relative to their threshold but rather returns the first one it encounters that fits the criteria.

Example Scenario:
Let's say there are three OperatorDelegators with the following settings:

totalTVL = 10,000 units
operatorDelegatorAllocations = [200, 300, 500] basis points (2%, 3%, and 5% respectively)
tvls = [150, 250, 450] units for each delegator
Calculation and Decision Process:
Threshold for Delegator 1: 
200
×
10
,
000
100
×
100
=
200
100×100
200×10,000
​
 =200 units
Threshold for Delegator 2: 
300
×
10
,
000
100
×
100
=
300
100×100
300×10,000
​
 =300 units
Threshold for Delegator 3: 
500
×
10
,
000
100
×
100
=
500
100×100
500×10,000
​
 =500 units
Given the actual tvls:

Delegator 1's TVL = 150 units, which is below the threshold of 200 units.
Delegator 2's TVL = 250 units, which is also below the threshold of 300 units.
Delegator 3's TVL = 450 units, which is below the threshold of 500 units.
Outcome:
As the function iterates, it finds that Delegator 1's TVL of 150 units is below its threshold of 200 units. Since Delegator 1 is the first to meet the criteria of having a TVL below the threshold, the function will return Delegator 1 immediately, without evaluating the remaining delegators.
Delegators 2 and 3, despite also having TVLs below their respective thresholds, are not considered because the function stops checking once the first qualifying delegator is found.

### Impact 
The first-match approach may inadvertently favor certain delegators that are consistently earlier in the list. This could introduce a ``bias in selection``, disadvantaging other ``delegators`` . Without considering the full context of each delegator’s current load and capabilities, there's a risk of creating imbalances where some delegators are overwhelmed while others are underutilized. This can lead to inefficiencies and increased wear on certain parts of the system.

```solidity
FILE: 2024-04-renzo/contracts/RestakeManager.sol

 // Otherwise, find the operator delegator with TVL below the threshold
        uint256 tvlLength = tvls.length;
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
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L375-L384

### Recommended Mitigation
Modify the selection algorithm to assess all delegators, calculating and comparing the difference between their current TVL and the threshold, and selecting the one with the greatest difference.

##

## [L-5] Replay Attacks and Stale Authorizations Due to Non-Expiring Signatures

Suppose a signature used in the transaction does not include an expiry time. This omission can make it vulnerable to replay attacks where the same transaction (signature) could be submitted repeatedly, potentially leading to unauthorized staking of ETH.

If the signature does not expire, it could be used long after the user’s intentions have changed. For example, a user might initially authorize a staking transaction but later, due to changing market conditions or personal preferences, wish to retract this authorization.

### Impact

- The repeated staking can cause financial discrepancies, allocate resources improperly, or disrupt the intended distribution and management of staked assets.

- The smart contract would still process the transaction based on the old, now potentially unwanted, authorization, leading to mismanagement of user funds or assets against their current wishes.

```solidity
FILE: 2024-04-renzo/contracts/RestakeManager.sol

/// @dev Called by the deposit queue to stake ETH to a validator
    /// Only callable by the deposit queue
    function stakeEthInOperatorDelegator(
        IOperatorDelegator operatorDelegator,
        bytes calldata pubkey,
        bytes calldata signature,
        bytes32 depositDataRoot
    ) external payable onlyDepositQueue {
        // Verify the OD is in the list
        bool found = false;
        uint256 odLength = operatorDelegators.length;
        for (uint256 i = 0; i < odLength; ) {
            if (operatorDelegators[i] == operatorDelegator) {
                found = true;
                break;
            }

            unchecked {
                ++i;
            }
        }
        if (!found) revert NotFound();

        // Call the OD to stake the ETH
        operatorDelegator.stakeEth{ value: msg.value }(pubkey, signature, depositDataRoot);
    }

```
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L618-L643

##

## [L-] Risk of Imbalanced Load Distribution Due to Default Selection Bias ``chooseOperatorDelegatorForDeposit()`` function 

### Impact

- There could be a consistent bias towards the first OperatorDelegator, especially in scenarios where frequently all OperatorDelegators exceed their respective thresholds. This can create an uneven distribution of deposits, potentially overloading the first OperatorDelegator.

- If the first OperatorDelegator has significantly different operational characteristics or capacities compared to others, consistently defaulting to it might lead to inefficiencies or increased risks of failure.

- The function does not balance the load among OperatorDelegators effectively if it always falls back to the first. This could defeat the purpose of having multiple OperatorDelegators, which is typically to distribute workload and reduce risks.

```solidity
FILE: 2024-04-renzo/contracts/RestakeManager.sol

/// @dev Picks the OperatorDelegator with the TVL below the threshold or returns the first one in the list
    /// @return The OperatorDelegator to use
    function chooseOperatorDelegatorForDeposit(
        uint256[] memory tvls,
        uint256 totalTVL
    ) public view returns (IOperatorDelegator) {
        // Ensure OperatorDelegator list is not empty
        if (operatorDelegators.length == 0) revert NotFound();

        // If there is only one operator delegator, return it
        if (operatorDelegators.length == 1) {
            return operatorDelegators[0];
        }

        // Otherwise, find the operator delegator with TVL below the threshold
        uint256 tvlLength = tvls.length;
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

        // Default to the first operator delegator
        return operatorDelegators[0];
    }


```
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L360-L393





 



 






## A. Reentrancy Vulnerability in `_withdraw` Function
[XERC20Lockbox.sol#L125-L136](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L125-L136)
In the `_withdraw` function, the contract emits a `Withdraw` event, burns the specified amount of XERC20 tokens from the caller's balance, and then interacts with external contracts to transfer tokens or native assets. However, the interaction step occurs before the effect step, which violates the Check-Effect-Interact (CEI) pattern, potentially leading to a reentrancy vulnerability.
### Impact:
This vulnerability could enable malicious actors to exploit reentrancy attacks. By calling back into the contract before the effect step (burning tokens) is completed, attackers could manipulate the contract's state, leading to unexpected behavior or loss of funds.

### Mitigation:

To address this vulnerability, we need to ensure that interactions with external contracts occur after completing the effect step (burning tokens). This sequence helps prevent reentrancy attacks by updating the contract's state before interacting with external entities.


## B. Division by Nearly Zero Denominator in `calculateMintAmount` Function
[RenzoOracle.sol#L123-L149](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol#L123-L149)
In the `calculateMintAmount` function, there is a potential vulnerability arising from the division operation used to calculate the new supply of ezETH tokens after a deposit. The relevant code snippet is as follows:
```solidity
uint256 newEzETHSupply = (_existingEzETHSupply * SCALE_FACTOR) /
            (SCALE_FACTOR - inflationPercentaage);
```
This division operation poses a risk when `inflationPercentaage` approaches or equals 100%. In such cases, the denominator (`SCALE_FACTOR - inflationPercentaage`) tends towards zero. Division by nearly zero can lead to arithmetic overflow or underflow, potentially compromising the integrity and functionality of the contract.

### Impact:
The impact of this vulnerability is significant as it could result in unexpected behavior or even contract failure. Attackers could potentially exploit this vulnerability to manipulate the minting process or disrupt the stability of the system.

### Mitigation:
To mitigate this vulnerability, it is crucial to add a safeguard to prevent division by nearly zero. One effective mitigation strategy is to include a check to ensure that the denominator is not close to zero before performing the division operation. This can be achieved by adding an if statement or using a require statement to validate the inflation percentage.
```solidity
if (inflationPercentaage >= SCALE_FACTOR) {
    // Handle the scenario where the inflation percentage is too high
    revert InflationTooHigh();
}

uint256 newEzETHSupply = (_existingEzETHSupply * SCALE_FACTOR) /
            (SCALE_FACTOR - inflationPercentaage);
```
## C. Lack of Secure Fee Transfer Mechanism
[DepositQueue.sol#L163-L183](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol#L163-L183)
The code snippet within the `DepositQueue.sol` contract attempts to transfer fees to an external address specified by `feeAddress` using a low-level call:
```solidity
(bool success, ) = feeAddress.call{ value: feeAmount }("");
if (!success) revert TransferFailed();

```
This mechanism utilizes a low-level call to transfer funds, which can introduce security risks. Low-level calls are susceptible to reentrancy attacks and other vulnerabilities, especially if the `feeAddress` is a smart contract with a fallback function. If the fallback function contains arbitrary or malicious logic, it could exploit vulnerabilities such as reentrancy or unexpected state changes, leading to unauthorized fund transfers or manipulation of contract state.

### Impact:
The lack of a secure fee transfer mechanism exposes the contract to potential attacks, including reentrancy exploits or unintended state modifications, which could result in loss of funds or manipulation of contract state.

### Mitigation:
To address this issue, consider using a safer and more robust method for fee transfers. Instead of using low-level calls, implement withdrawal patterns that separate the fee transfer logic from other contract operations. One effective mitigation strategy is to adopt the "Pull over Push" pattern, where external entities initiate the withdrawal process by pulling funds from the contract using a designated withdrawal function.
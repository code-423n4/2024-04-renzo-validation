## Medium Findings

|    | Issue | Instances |
|----|-------|:---------:|
| [M-01] | Unchecked Return Values of the `approve()` Function | 1 |
| [M-02] | Unsafe use of `transfer()/transferFrom()` with IERC20 | 1 |
| [M-03] | Gas griefing/theft is possible on an unsafe external call | 4 |
| [M-04] | Inconsistent Behavior of `block.number` Across Layer-2 Solutions | 1 |

## Low Findings

|    | Issue | Instances |
|----|-------|:---------:|
| [L-01] | Possible Vulnerability to Fee-On-Transfer Accounting Issues | 8 |
| [L-02] | Use `increaseAllowance`/`decreaseAllowance` instead of `approve`/`safeApprove` | 8 |
| [L-03] | Low Level Calls to Custom Addresses | 1 |
| [L-04] | Unchecked Return Values of `transfer()/transferFrom()` | 1 |
| [L-05] | Unused/empty `receive()/fallback()` function | 2 |
| [L-06] | `decimals()` is not part of the ERC20 standard | 4 |
| [L-07] | Large transfers may not work with some `ERC20` tokens | 10 |
| [L-08] | Use `call()` rather than `transfer()/send()` to sent ether | 4 |
| [L-09] | Code does not follow the best practice of check-effects-interaction | 4 |
| [L-10] | Initializers can be front-run | 18 |
| [L-11] | Prefer skip over `revert` model in loops | 7 |
| [L-12] | Unbounded loop may run out of gas | 11 |
| [L-13] | Consider checking that transfer to `address(this)` is disabled | 4 |
| [L-14] | Possible reentrancy with callback on transfer tokens | 1 |
| [L-15] | Possible division by 0 is not prevented | 2 |
| [L-16] | Solidity version `0.8.20` may not work on other chains due to `PUSH0` | 10 |
| [L-17] | Initializer missing `address(0)` Check before assignment | 6 |
| [L-18] | Missing checks for `address(0)` when updating state variables | 1 |
| [L-19] | Risk of Permanently Locked Ether | 1 |
| [L-20] | Risk of Renouncing Ownership While System is Paused | 3 |
| [L-21] | Input Validation Missing in Setter Functions | 1 |
| [L-22] | Avoid using `tx.origin` | 6 |
| [L-23] | Consider the case where `totalSupply` is zero | 3 |
| [L-24] | File allows a version of solidity that is susceptible to .selector-related optimizer bug | 3 |
| [L-25] | Missing checks for state variable assignments | 2 |
| [L-26] | Large approvals may not work with some ERC20 tokens | 6 |
| [L-27] | Missing checks in constructor/initialize | 7 |
| [L-28] | Owner can renounce Ownership | 5 |
| [L-29] | State variables are not limited to a reasonable range | 1 |
| [L-30] | Missing contract-existence checks before low-level calls | 4 |
| [L-31] | External calls in an unbounded loop may revert | 20 |
| [L-32] | Input Arrays Lengths Not Checked | 8 |
| [L-33] | Critical functions should be a two step procedure | 11 |
| [L-34] | `onlyOwner` functions not accessible if `owner` renounces ownership | 20 |
| [L-35] | safeApprove is deprecated | 10 |
| [L-36] | Loss of precision on division | 20 |
| [L-37] | Unbounded Gas Consumption on External Calls | 5 |
| [L-38] | Lack of two-step update for updating protocol addresses | 13 |
| [L-39] | Some `ERC20` can revert on a zero value `transfer` | 1 |
| [L-40] | Functions calling contracts with transfer hooks are missing reentrancy guards | 9 |
| [L-41] | Missing `__gap[50]` storage variable in Upgradeable contracts | 13 |
| [L-42] | Use of ownership with a single step rather than double | 7 |
| [L-43] | Consider using OpenZeppelin's SafeCast for any casting | 8 |
| [L-44] | Using Low-Level Call for Transfers | 4 |


## Medium Findings Details

### [M-01] Unchecked Return Values of the `approve()` Function

The unchecked return value of the `approve()` method can potentially cause transaction failures to go unnoticed in your contract. 

Some IERC20 token implementations utilize boolean return values to indicate transaction failures, instead of relying on the `revert()` function. If the return value of the `approve()` method isn't appropriately verified, transactions may seemingly proceed even when the necessary token approvals have not been appropriately executed.

As a safeguard, it is crucial to consistently verify the return values of these methods. This precaution helps to ensure the correct execution of token approvals, maintaining the integrity of transaction operations and reducing the risk of unnoticed transaction failures.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Deposits/DepositQueue.sol

268: token.approve(address(restakeManager), balance - feeAmount);
```
[268](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L268)
</details>

### [M-02] Unsafe use of `transfer()/transferFrom()` with IERC20

Not all tokens adhere strictly to the ERC20 standard, even if they are largely accepted as compliant.
A prominent example is Tether (USDT) on L1, where the transfer() and transferFrom() functions do not return booleans, contrary to the standard's requirements.
Instead, these functions provide no return value at all.
This discrepancy can lead to issues when such tokens are cast to IERC20: their non-standard function signatures can cause calls to revert.
As a safer alternative, consider using OpenZeppelin's SafeERC20 library, which offers safeTransfer() and safeTransferFrom() functions to address these incompatibilities.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Withdraw/WithdrawQueue.sol

305: IERC20(_withdrawRequest.collateralToken).transfer(
                msg.sender,
                _withdrawRequest.amountToRedeem
            );
```
[305](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L305)
</details>

### [M-03] Gas griefing/theft is possible on an unsafe external call

A low-level call will copy any amount of bytes to local memory. When bytes are copied from returndata to memory, the memory expansion cost is paid.

This means that when using a standard solidity call, the callee can 'returnbomb' the caller, imposing an arbitrary gas cost.

Because this gas is paid by the caller and in the caller's context, it can cause the caller to run out of gas and halt execution.

Consider replacing all unsafe `call` with `excessivelySafeCall` from this [repository](https://github.com/nomad-xyz/ExcessivelySafeCall).

<details>
<summary><i>4 issue instances in 4 files:</i></summary>

```solidity
File: contracts/TimelockController.sol

369: (bool success, ) = target.call{ value: value }(data);
```
[369](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L369)

```solidity
File: contracts/Rewards/RewardHandler.sol

68: (bool success, ) = rewardDestination.call{ value: balance }("");
```
[68](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L68)

```solidity
File: contracts/Deposits/DepositQueue.sol

168: (bool success, ) = feeAddress.call{ value: feeAmount }("");
```
[168](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L168)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

520: (bool success, ) = destination.call{ value: remainingAmount }("");
```
[520](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L520)
</details>

### [M-04] Inconsistent Behavior of `block.number` Across Layer-2 Solutions

`block.number` can behave differently across various Layer-2 (L2) solutions. 
For instance, on Optimism, `block.number` corresponds to the L2 block number, but on Arbitrum, it matches the Layer-1 (L1) block number.
Moreover, L2 block numbers often increase more frequently than L1 block numbers.
This discrepancy can introduce inconsistencies when timing events or performing cross-chain operations.
Consider using a consistent clock mechanism, like the one provided by OpenZeppelin's Governor contract from version 4.9, to avoid potential issues.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Delegation/OperatorDelegator.sol

247: block.number,
```
[247](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L247)
</details>


## Low Findings Details

### [L-01] Possible Vulnerability to Fee-On-Transfer Accounting Issues

Contracts transfer funds using the `transferFrom()` function but do not verify that the actual number of tokens received matches the input amount to the transfer.
This could lead to accounting issues if the token involves a fee on transfer.
An attacker might exploit latent funds to get a free credit.
To prevent this, consider checking the balance before and after the transfer and use the difference as the actual amount received.

<details>
<summary><i>8 issue instances in 6 files:</i></summary>

```solidity
File: contracts/Withdraw/WithdrawQueue.sol

305: IERC20(_withdrawRequest.collateralToken).transfer(
                msg.sender,
                _withdrawRequest.amountToRedeem
            );
```
[305](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L305)

```solidity
File: contracts/RestakeManager.sol

540: _collateralToken.safeTransferFrom(msg.sender, address(this), _amount);
661: _token.safeTransferFrom(msg.sender, address(this), _amount);
```
[540](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L540) | [661](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L661)

```solidity
File: contracts/Deposits/DepositQueue.sol

140: IERC20(_asset).safeTransferFrom(msg.sender, address(this), _amount);
```
[140](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L140)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

151: token.safeTransferFrom(msg.sender, address(this), tokenAmount);
```
[151](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L151)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

490: IERC20(_token).safeTransfer(_to, _amount);
```
[490](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L490)

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol

306: IERC20(_token).safeTransfer(_to, _amount);
```
[295](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L295) | [306](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L306)
</details>

### [L-02] Use `increaseAllowance`/`decreaseAllowance` instead of `approve`/`safeApprove`

Changing an allowance with `approve` brings the risk that someone may use both the old and the new allowance by unfortunate transaction ordering. Refer to [ERC20 API: An Attack Vector on the Approve/TransferFrom Methods](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit#heading=h.m9fhqynw2xvt)

It is recommended to use the `increaseAllowance`/`decreaseAllowance` to avoid ths problem.

<details>
<summary><i>8 issue instances in 4 files:</i></summary>

```solidity
File: contracts/RestakeManager.sol

552: _collateralToken.safeApprove(address(depositQueue), bufferToFill);
559: _collateralToken.safeApprove(address(operatorDelegator), _amount);
664: _token.safeApprove(address(operatorDelegator), _amount);
```
[552](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L552) | [559](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L559) | [664](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L664)

```solidity
File: contracts/Deposits/DepositQueue.sol

142: IERC20(_asset).safeApprove(address(withdrawQueue), _amount);
268: token.approve(address(restakeManager), balance - feeAmount);
```
[142](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L142) | [268](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L268)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

164: _token.safeApprove(address(strategyManager), _tokenAmount);
```
[164](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L164)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

369: depositToken.safeApprove(address(connext), _amountIn);
429: collateralToken.safeApprove(address(connext), balance);
```
[369](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L369) | [429](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L429)
</details>

### [L-03] Low Level Calls to Custom Addresses

Contracts should avoid making low-level calls to custom addresses, especially if these calls are based on address parameters in the function. 
Such behavior can lead to unexpected execution of untrusted code. Instead, consider using Solidity's high-level function calls or contract interactions.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/TimelockController.sol

369: (bool success, ) = target.call{ value: value }(data);
```
[369](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L369)
</details>

### [L-04] Unchecked Return Values of `transfer()/transferFrom()`

`transfer()` or `transferFrom()` methods without checking their return values.
Some IERC20 implementations use these boolean return values to signal transaction failures instead of using revert().
Not checking these values could allow transactions to proceed without the intended token transfers.
It is recommended to always check the return values of these methods.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Withdraw/WithdrawQueue.sol

305: IERC20(_withdrawRequest.collateralToken).transfer(
                msg.sender,
                _withdrawRequest.amountToRedeem
            );
```
[305](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L305)
</details>

### [L-05] Unused/empty `receive()/fallback()` function

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))).

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

542: receive() external payable
```
[542](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L542)

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol

313: receive() external payable
```
[313](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L313)
</details>

### [L-06] `decimals()` is not part of the ERC20 standard

The `decimals` function is not part of the `ERC20` standard, and they may revert with some tokens if the don't also extend the `IERC20Metadata` interface.

<details>
<summary><i>4 issue instances in 2 files:</i></summary>

```solidity
File: contracts/RestakeManager.sol

231: if (IERC20Metadata(address(_newCollateralToken)).decimals() != 18)
234: IERC20Metadata(address(_newCollateralToken)).decimals()
```
[231](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L231) | [234](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L234)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

105: uint8 decimals = IERC20MetadataUpgradeable(address(_depositToken)).decimals();
109: decimals = IERC20MetadataUpgradeable(address(_collateralToken)).decimals();
```
[105](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L105) | [109](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L109)
</details>

### [L-07] Large transfers may not work with some `ERC20` tokens

Some `IERC20` implementations (e.g `UNI`, `COMP`)
            
- [COMP (Compound Protocol)](https://github.com/compound-finance/compound-protocol/blob/a3214f67b73310d547e00fc578e8355911c9d376/contracts/Governance/Comp.sol#L115-L142)
- [UNI (Uniswap)](https://github.com/Uniswap/governance/blob/eabd8c71ad01f61fb54ed6945162021ee419998e/contracts/Uni.sol#L209-L236)
            
may fail if the valued `transferred` is larger than `uint96`.

<details>
<summary><i>10 issue instances in 6 files:</i></summary>

```solidity
File: contracts/Withdraw/WithdrawQueue.sol

197: IERC20(_asset).safeTransferFrom(msg.sender, address(this), _amount)
305: IERC20(_withdrawRequest.collateralToken).transfer(
                msg.sender,
                _withdrawRequest.amountToRedeem
            )
```
[197](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L197) | [305](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L305)

```solidity
File: contracts/RestakeManager.sol

540: _collateralToken.safeTransferFrom(msg.sender, address(this), _amount)
661: _token.safeTransferFrom(msg.sender, address(this), _amount)
```
[540](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L540) | [661](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L661)

```solidity
File: contracts/Deposits/DepositQueue.sol

140: IERC20(_asset).safeTransferFrom(msg.sender, address(this), _amount)
262: IERC20(token).safeTransfer(feeAddress, feeAmount)
```
[140](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L140) | [262](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L262)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

151: token.safeTransferFrom(msg.sender, address(this), tokenAmount)
```
[151](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L151)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

214: depositToken.safeTransferFrom(msg.sender, address(this), _amountIn)
490: IERC20(_token).safeTransfer(_to, _amount)
```
[214](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L214) | [490](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L490)

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol

306: IERC20(_token).safeTransfer(_to, _amount)
```
[306](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L306)
</details>

### [L-08] Use `call()` rather than `transfer()/send()` to sent ether

The use of `transfer()/send()` is heavily frowned upon because it can lead to the locking of funds. The `transfer()/send()` call requires that the recipient is either an EOA account, or is a contract that has a payable callback.
For the contract case, the `transfer()/send()` call only provides 2300 gas for the contract to complete its operations.
This means the following cases can cause the transfer to fail:

- Contract does not have a payable callback
- Contract's payable callback spends more than 2300 gas (which is only enough to emit something)
- The destination is a smart contract but that smart contract is called via an intermediate proxy contract increasing the case requirements to more than 2300 gas units. A further example of unknown destination complexity is that of a multisig wallet that as part of its operation uses more than 2300 gas units.
- Future changes or forks in Ethereum result in higher gas fees than transfer provides. The .`transfer()/send()` creates a hard dependency on 2300 gas units being appropriate now and into the future.

<details>
<summary><i>4 issue instances in 4 files:</i></summary>

```solidity
File: contracts/Withdraw/WithdrawQueue.sol

303: payable(msg.sender).transfer(_withdrawRequest.amountToRedeem)
```
[303](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L303)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

485: payable(tx.origin).send(gasRefund)
```
[485](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L485)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

479: payable(_to).transfer(_amount)
```
[479](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L479)

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol

295: payable(_to).transfer(_amount)
```
[295](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L295)
</details>

### [L-09] Code does not follow the best practice of check-effects-interaction

Code should follow the best-practice of [CEI](https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-11-coding-patterns/topic/checks-effects-interactions/), where state variables are updated before any external calls are made. Doing so prevents a large class of reentrancy bugs.

<details>
<summary><i>4 issue instances in 3 files:</i></summary>

```solidity
File: contracts/Withdraw/WithdrawQueue.sol

/// @audit - State change after external call: `IERC20(address(ezETH)).safeTransferFrom(msg.sender, address(this), _amount)`
253: claimReserve[_assetOut] += amountToRedeem
```
[253](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L253)

```solidity
File: contracts/Deposits/DepositQueue.sol

/// @audit - State change after external call: `feeAddress.call{value: feeAmount}`
179: totalEarned[address(0x0)] = totalEarned[address(0x0)] + remainingRewards
/// @audit - State change after external call: `IERC20(token).safeTransfer(feeAddress, feeAmount)`
272: totalEarned[address(token)] = totalEarned[address(token)] + balance - feeAmount
```
[179](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L179) | [272](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L272)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

/// @audit - State change after external call: `payable(tx.origin).send(gasRefund)`
489: adminGasSpentInWei[tx.origin] -= gasRefund
```
[489](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L489)
</details>

### [L-10] Initializers can be front-run

Initializers could be vulnerable to front-running attacks.
This might allow an attacker to set their own values, take ownership of the contract, or force a redeployment in the best-case scenario.
Be cautious of potential front-running risks in the following instances found in the code.

<details>
<summary><i>18 issue instances in 18 files:</i></summary>

```solidity
File: contracts/token/EzEthToken.sol

33: function initialize(IRoleManager _roleManager) public initializer
```
[33](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L33)

```solidity
File: contracts/Withdraw/WithdrawQueue.sol

64: function initialize(
        IRoleManager _roleManager,
        IRestakeManager _restakeManager,
        IEzEthToken _ezETH,
        IRenzoOracle _renzoOracle,
        uint256 _coolDownPeriod,
        TokenWithdrawBuffer[] calldata _withdrawalBufferTarget
    ) external initializer
```
[64](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L64)

```solidity
File: contracts/Rewards/RewardHandler.sol

38: function initialize(IRoleManager _roleManager, address _rewardDestination) public initializer
```
[38](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L38)

```solidity
File: contracts/RestakeManager.sol

101: function initialize(
        IRoleManager _roleManager,
        IEzEthToken _ezETH,
        IRenzoOracle _renzoOracle,
        IStrategyManager _strategyManager,
        IDelegationManager _delegationManager,
        IDepositQueue _depositQueue
    ) public initializer
```
[101](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L101)

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol

17: function initialize(
        IRestakeManager _restakeManager,
        IERC20Upgradeable _ezETHToken
    ) public initializer
```
[17](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L17)

```solidity
File: contracts/Permissions/RoleManager.sol

22: function initialize(address roleManagerAdmin) public initializer
```
[22](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L22)

```solidity
File: contracts/Oracle/RenzoOracle.sol

44: function initialize(IRoleManager _roleManager) public initializer
```
[44](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L44)

```solidity
File: contracts/Oracle/Mantle/METHShim.sol

23: function initialize(IMethStaking _methStaking) public initializer
```
[23](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L23)

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol

23: function initialize(IStakedTokenV2 _wBETHToken) public initializer
```
[23](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L23)

```solidity
File: contracts/Deposits/DepositQueue.sol

74: function initialize(IRoleManager _roleManager) public initializer
```
[74](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L74)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

74: function initialize(
        IRoleManager _roleManager,
        IStrategyManager _strategyManager,
        IRestakeManager _restakeManager,
        IDelegationManager _delegationManager,
        IEigenPodManager _eigenPodManager
    ) external initializer
```
[74](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L74)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol

61: function initialize(
        string memory _name,
        string memory _symbol,
        address _factory
    ) public initializer
```
[61](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L61)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol

54: function initialize(
        address _lockboxImplementation,
        address _xerc20Implementation
    ) public initializer
```
[54](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L54)

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol

35: function initialize(
        string memory _name,
        string memory _symbol,
        address _factory,
        address _l1Token,
        address _optimismBridge
    ) public initializer
```
[35](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L35)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

75: function initialize(
        uint256 _currentPrice,
        IERC20 _xezETH,
        IERC20 _depositToken,
        IERC20 _collateralToken,
        IConnext _connext,
        bytes32 _swapKey,
        address _receiver,
        uint32 _bridgeDestinationDomain,
        address _bridgeTargetAddress,
        IRenzoOracleL2 _oracle
    ) public initializer
```
[75](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L75)

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol

23: function initialize(AggregatorV3Interface _oracle) public initializer
```
[23](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L23)

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol

70: function initialize(
        IERC20 _ezETH,
        IERC20 _xezETH,
        IRestakeManager _restakeManager,
        IERC20 _wETH,
        IXERC20Lockbox _xezETHLockbox,
        IConnext _connext,
        IRouterClient _linkRouterClient,
        IRateProvider _rateProvider,
        LinkTokenInterface _linkToken,
        IRoleManager _roleManager
    ) public initializer
```
[70](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L70)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol

44: function initialize(address _xerc20, address _erc20, bool _isNative) public initializer
```
[44](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L44)
</details>

### [L-11] Prefer skip over `revert` model in loops

When dealing with array operations in Solidity, it's often wiser to opt for skipping over reverting.
Instead of halting the entire transaction when a condition isn't met, suggests simply moving on to the next array index.
The reason behind this choice is to reduce the risk of malicious actors intentionally introducing array objects that fail conditional checks within loops, causing group operations to fail.
Unless there's a compelling security or logical justification for reverting, it's recommended to embrace the skip-over approach for a more robust codebase.

<details>
<summary><i>7 issue instances in 4 files:</i></summary>

```solidity
File: contracts/Withdraw/WithdrawQueue.sol

92: revert InvalidZeroInput()
112: revert InvalidZeroInput()
```
[92](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L92) | [112](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L112)

```solidity
File: contracts/RestakeManager.sol

139: revert AlreadyAdded()
224: revert AlreadyAdded()
```
[139](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L139) | [224](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L224)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

218: revert InvalidZeroInput()
278: revert InvalidZeroInput()
```
[218](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L218) | [278](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L278)

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol

238: revert NotEnoughBalance(linkToken.balanceOf(address(this)), fees)
```
[238](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L238)
</details>

### [L-12] Unbounded loop may run out of gas

Unbounded loops in smart contracts pose a risk because they iterate over an unknown number of elements, potentially consuming all available gas for a transaction.
This can result in unintended transaction failures. Gas consumption increases linearly with the number of iterations, and if it surpasses the gas limit, the transaction reverts, wasting the gas spent.
To mitigate this, developers should either set a maximum limit on loop iterations.

<details>
<summary><i>11 issue instances in 4 files:</i></summary>

```solidity
File: contracts/Withdraw/WithdrawQueue.sol

88: for (uint256 i = 0; i < _withdrawalBufferTarget.length; )
110: for (uint256 i = 0; i < _newBufferTarget.length; )
```
[88](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L88) | [110](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L110)

```solidity
File: contracts/TimelockController.sol

107: for (uint256 i = 0; i < proposers.length; ++i)
113: for (uint256 i = 0; i < executors.length; ++i)
272: for (uint256 i = 0; i < targets.length; ++i)
355: for (uint256 i = 0; i < targets.length; ++i)
```
[107](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L107) | [113](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L113) | [272](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L272) | [355](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L355)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

209: for (uint256 i; i < tokens.length; )
277: for (uint256 i; i < tokens.length; )
381: for (uint256 i = 0; i < validatorFields.length; )
```
[209](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L209) | [277](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L277) | [381](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L381)

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol

218: for (uint256 i = 0; i < _destinationParam.length; )
264: for (uint256 i = 0; i < _connextDestinationParam.length; )
```
[218](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L218) | [264](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L264)
</details>

### [L-13] Consider checking that transfer to `address(this)` is disabled

Functions allowing users to specify the receiving address should check whether the referenced address is not `address(this)`.

This would prevent tokens to remain lock inside the contract due to an incorrect `transfer` or `mint`.

<details>
<summary><i>4 issue instances in 4 files:</i></summary>

```solidity
File: contracts/token/EzEthToken.sol

42: _mint(to, amount)
```
[42](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L42)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol

357: _mint(_user, _amount)
```
[357](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L357)

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol

65: XERC20.mint(_to, _amount)
```
[65](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L65)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol

150: XERC20.mint(_to, _amount)
```
[150](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L150)
</details>

### [L-14] Possible reentrancy with callback on transfer tokens

The following functions don't apply the CEI pattern. It's possible to reenter after the transfer if the token has some kind of callback functionality (e.g. ERC777/ERC1155).

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Deposits/DepositQueue.sol

/// @audit - State change after transfer: `totalEarned[address(token)] = totalEarned[address(token)] + balance - feeAmount`
262: IERC20(token).safeTransfer(feeAddress, feeAmount)
```
[262](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L262)
</details>

### [L-15] Possible division by 0 is not prevented

These functions can be called with 0 value in the input and this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol

/// @audit totalSupply - can be zero
40: return (10 ** 18 * totalTVL) / totalSupply;
```
[40](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L40)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

/// @audit lastPric - can be zero
253: uint256 xezETHAmount = (1e18 * amountOut) / _lastPrice;
```
[253](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L253)
</details>

### [L-16] Solidity version `0.8.20` may not work on other chains due to `PUSH0`

In Solidity `0.8.20`'s compiler, the default target EVM version has been changed to Shanghai. This version introduces a new op code called `PUSH0`.

However, not all Layer 2 solutions have implemented this op code yet, leading to deployment failures on these chains. To overcome this problem, it is recommended to utilize an earlier EVM version.

<details>
<summary><i>10 issue instances in 10 files:</i></summary>

```solidity
File: contracts/token/EzEthToken.sol

2: pragma solidity ^0.8.9;
```
[2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L2)

```solidity
File: contracts/TimelockController.sol

4: pragma solidity ^0.8.0;
```
[4](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L4)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol

2: pragma solidity >=0.8.4 <0.9.0;
```
[2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L2)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol

2: pragma solidity >=0.8.4 <0.9.0;
```
[2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L2)

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol

2: pragma solidity >=0.8.4 <0.9.0;
```
[2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L2)

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol

2: pragma solidity >=0.8.4 <0.9.0;
```
[2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L2)

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol

2: pragma solidity ^0.8.13;
```
[2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L2)

```solidity
File: contracts/Bridge/Connext/libraries/LibConnextStorage.sol

2: pragma solidity ^0.8.0;
```
[2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/libraries/LibConnextStorage.sol#L2)

```solidity
File: contracts/Bridge/Connext/libraries/TokenId.sol

2: pragma solidity ^0.8.0;
```
[2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/libraries/TokenId.sol#L2)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol

2: pragma solidity >=0.8.4 <0.9.0;
```
[2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L2)
</details>

### [L-17] Initializer missing `address(0)` Check before assignment

The initializer does not include a check for `address(0)` when initializing state variables that hold addresses.
Initializing a state variable with `address(0)` can lead to unintended behavior and vulnerabilities in the contract, 
such as sending funds to an inaccessible address. 
It is recommended to include a validation step to ensure that address parameters are not set to `address(0)`.

<details>
<summary><i>6 issue instances in 6 files:</i></summary>

```solidity
File: contracts/RestakeManager.sol

/// @audit `_roleManager` has lack of `address(0)` check before use
/// @audit `_ezETH` has lack of `address(0)` check before use
/// @audit `_renzoOracle` has lack of `address(0)` check before use
/// @audit `_strategyManager` has lack of `address(0)` check before use
/// @audit `_delegationManager` has lack of `address(0)` check before use
/// @audit `_depositQueue` has lack of `address(0)` check before use
101: function initialize(
        IRoleManager _roleManager,
        IEzEthToken _ezETH,
        IRenzoOracle _renzoOracle,
        IStrategyManager _strategyManager,
        IDelegationManager _delegationManager,
        IDepositQueue _depositQueue
    ) public initializer {
```
[101](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L101)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol

/// @audit `_factory` has lack of `address(0)` check before use
61: function initialize(
        string memory _name,
        string memory _symbol,
        address _factory
    ) public initializer {
```
[61](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L61)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol

/// @audit `_lockboxImplementation` has lack of `address(0)` check before use
/// @audit `_xerc20Implementation` has lack of `address(0)` check before use
54: function initialize(
        address _lockboxImplementation,
        address _xerc20Implementation
    ) public initializer {
```
[54](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L54)

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol

/// @audit `_factory` has lack of `address(0)` check before use
/// @audit `_l1Token` has lack of `address(0)` check before use
/// @audit `_optimismBridge` has lack of `address(0)` check before use
35: function initialize(
        string memory _name,
        string memory _symbol,
        address _factory,
        address _l1Token,
        address _optimismBridge
    ) public initializer {
```
[35](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L35)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

/// @audit `_receiver` has lack of `address(0)` check before use
/// @audit `_oracle` has lack of `address(0)` check before use
75: function initialize(
        uint256 _currentPrice,
        IERC20 _xezETH,
        IERC20 _depositToken,
        IERC20 _collateralToken,
        IConnext _connext,
        bytes32 _swapKey,
        address _receiver,
        uint32 _bridgeDestinationDomain,
        address _bridgeTargetAddress,
        IRenzoOracleL2 _oracle
    ) public initializer {
```
[75](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L75)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol

/// @audit `_xerc20` has lack of `address(0)` check before use
/// @audit `_erc20` has lack of `address(0)` check before use
44: function initialize(address _xerc20, address _erc20, bool _isNative) public initializer {
```
[44](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L44)
</details>

### [L-18] Missing checks for `address(0)` when updating state variables

Check for zero-address to avoid the risk of setting `address(0)` for state variables after an update.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol

123: lockbox = _lockbox;
```
[123](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L123)
</details>

### [L-19] Risk of Permanently Locked Ether

When Ether is mistakenly sent to a contract without a means of retrieval, it becomes irrevocably locked.
Incidents of accidental Ether transfers have been observed even in high-profile projects, potentially leading to significant financial setbacks.
To enhance contract resilience, it's recommended to incorporate an "Ether recovery" function to serve as a protective measure against unintended Ether lockups.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol

313: receive() external payable {}
```
[313](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L313)
</details>

### [L-20] Risk of Renouncing Ownership While System is Paused

If the owner or a role-holder renounce or loses their role/ownership while the system is paused, it can lead to user assets becoming permanently locked.

<details>
<summary><i>3 issue instances in 3 files:</i></summary>

```solidity
File: contracts/Withdraw/WithdrawQueue.sol

139: function pause() external onlyWithdrawQueueAdmin {
```
[139](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L139)

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol

121: function pause() external onlyOwner {
```
[121](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L121)

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol

125: function pause() external onlyOwner {
```
[125](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L125)
</details>

### [L-21] Input Validation Missing in Setter Functions

Setter functions are utilized to update the state variables of a contract.
It is critical to ensure these functions have adequate input sanitization to prevent unwanted alterations or malicious attacks.
Without input validation, there's a potential risk of enabling vulnerabilities like overflow/underflow, unauthorized access, or insertion of invalid data.
Consider incorporating appropriate validation mechanisms, such as checking the range or type of inputs, to enhance the security of your contract.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/RestakeManager.sol

/// @audit `setMaxDepositTVL` function does not validate `_maxDepositTVL` input
215: function setMaxDepositTVL(uint256 _maxDepositTVL) external onlyRestakeManagerAdmin {
```
[215](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L215)
</details>

### [L-22] Avoid using `tx.origin`

The use of `tx.origin` can potentially make a contract vulnerable if an authorized account calls a malicious contract.

It is recommended to avoid using `tx.origin` as it can lead to easier impersonation of users via third party contracts.

<details>
<summary><i>6 issue instances in 1 files:</i></summary>

```solidity
File: contracts/Delegation/OperatorDelegator.sol

482: uint256 gasRefund = address(this).balance >= adminGasSpentInWei[tx.origin]
483: ? adminGasSpentInWei[tx.origin]
485: bool success = payable(tx.origin).send(gasRefund);
489: adminGasSpentInWei[tx.origin] -= gasRefund;
491: emit GasRefunded(tx.origin, gasRefund);
509: if (adminGasSpentInWei[tx.origin] > 0) {
```
[482](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L482) | [483](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L483) | [485](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L485) | [489](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L489) | [491](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L491) | [509](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L509)
</details>

### [L-23] Consider the case where `totalSupply` is zero

The following functions should handle the edge case where the `totalSupply` is zero, for example to avoid division by zero errors, as such errors may negatively impact the logic of these functions.

<details>
<summary><i>3 issue instances in 2 files:</i></summary>

```solidity
File: contracts/Withdraw/WithdrawQueue.sol

222: ezETH.totalSupply(),
```
[222](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L222)

```solidity
File: contracts/RestakeManager.sol

568: ezETH.totalSupply()
608: ezETH.totalSupply()
```
[568](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L568) | [608](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L608)
</details>

### [L-24] File allows a version of solidity that is susceptible to .selector-related optimizer bug

In solidity versions prior to 0.8.21, there is a legacy code generation [bug](https://soliditylang.org/blog/2023/07/19/missing-side-effects-on-selector-access-bug/) where if foo().selector is called, foo() doesn't actually get evaluated.
It is listed as low-severity, because projects usually use the contract name rather than a function call to look up the selector.

<details>
<summary><i>3 issue instances in 1 files:</i></summary>

```solidity
File: contracts/TimelockController.sol

417: return this.onERC721Received.selector;
430: return this.onERC1155Received.selector;
443: return this.onERC1155BatchReceived.selector;
```
[417](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L417) | [430](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L430) | [443](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L443)
</details>

### [L-25] Missing checks for state variable assignments

There are some missing checks in these functions, and this could lead to unexpected scenarios. Consider always adding a sanity check for state variables.

<details>
<summary><i>2 issue instances in 2 files:</i></summary>

```solidity
File: contracts/RestakeManager.sol

714: collateralTokenTvlLimits[_token] = _limit;
```
[714](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L714)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

111: tokenStrategyMapping[_token] = _strategy;
```
[111](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L111)
</details>

### [L-26] Large approvals may not work with some ERC20 tokens

Certain tokens, may fail if the valued passed to `approve` is larger than `uint96`.
- [COMP (Compound Protocol)](https://github.com/compound-finance/compound-protocol/blob/a3214f67b73310d547e00fc578e8355911c9d376/contracts/Governance/Comp.sol#L86-L98)
- [UNI (Uniswap)](https://github.com/Uniswap/governance/blob/eabd8c71ad01f61fb54ed6945162021ee419998e/contracts/Uni.sol#L149-L161)

<details>
<summary><i>6 issue instances in 3 files:</i></summary>

```solidity
File: contracts/RestakeManager.sol

552: _collateralToken.safeApprove(address(depositQueue), bufferToFill);
559: _collateralToken.safeApprove(address(operatorDelegator), _amount);
664: _token.safeApprove(address(operatorDelegator), _amount);
```
[552](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L552) | [559](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L559) | [664](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L664)

```solidity
File: contracts/Deposits/DepositQueue.sol

142: IERC20(_asset).safeApprove(address(withdrawQueue), _amount);
268: token.approve(address(restakeManager), balance - feeAmount);
```
[142](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L142) | [268](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L268)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

429: collateralToken.safeApprove(address(connext), balance);
```
[429](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L429)
</details>

### [L-27] Missing checks in constructor/initialize

There are some missing checks in these functions, and this could lead to unexpected scenarios. Consider always adding a sanity check for state variables.

<details>
<summary><i>7 issue instances in 6 files:</i></summary>

```solidity
File: contracts/TimelockController.sol

/// @audit `minDelay` is not validated before use
/// @audit `minDelay` is not validated before use
87: constructor(
        uint256 minDelay,
        address[] memory proposers,
        address[] memory executors,
        address admin
    ) {
```
[87](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L87)

```solidity
File: contracts/RestakeManager.sol

/// @audit `_roleManager` is not validated before use
/// @audit `_ezETH` is not validated before use
/// @audit `_renzoOracle` is not validated before use
/// @audit `_strategyManager` is not validated before use
/// @audit `_delegationManager` is not validated before use
/// @audit `_depositQueue` is not validated before use
101: function initialize(
        IRoleManager _roleManager,
        IEzEthToken _ezETH,
        IRenzoOracle _renzoOracle,
        IStrategyManager _strategyManager,
        IDelegationManager _delegationManager,
        IDepositQueue _depositQueue
    ) public initializer {
```
[101](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L101)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol

/// @audit `_name` passed to inherited constructor is not validated
/// @audit `_symbol` passed to inherited constructor is not validated
43: // constructor(string memory _name, string memory _symbol, address _factory) ERC20Upgradeable(_name, _symbol) ERC20PermitUpgradeable(_name) {
/// @audit `_name` is not validated before use
/// @audit `_symbol` is not validated before use
61: function initialize(
        string memory _name,
        string memory _symbol,
        address _factory
    ) public initializer {
```
[43](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L43) | [61](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L61)

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol

/// @audit `_name` is not validated before use
/// @audit `_symbol` is not validated before use
35: function initialize(
        string memory _name,
        string memory _symbol,
        address _factory,
        address _l1Token,
        address _optimismBridge
    ) public initializer {
```
[35](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L35)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

/// @audit `_oracle` is not validated before use
75: function initialize(
        uint256 _currentPrice,
        IERC20 _xezETH,
        IERC20 _depositToken,
        IERC20 _collateralToken,
        IConnext _connext,
        bytes32 _swapKey,
        address _receiver,
        uint32 _bridgeDestinationDomain,
        address _bridgeTargetAddress,
        IRenzoOracleL2 _oracle
    ) public initializer {
```
[75](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L75)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol

/// @audit `_isNative` is not validated before use
44: function initialize(address _xerc20, address _erc20, bool _isNative) public initializer {
```
[44](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L44)
</details>

### [L-28] Owner can renounce Ownership

Each of the following contracts implements or inherits the `renounceOwnership` function.
This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

<details>
<summary><i>5 issue instances in 5 files:</i></summary>

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol

11: import {
    OwnableUpgradeable
} from "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
```
[11](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L11)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

6: import {
    OwnableUpgradeable
} from "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
```
[6](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L6)

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol

5: import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol";
```
[5](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L5)

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol

8: import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol";
```
[8](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L8)

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol

6: import {
    OwnableUpgradeable
} from "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
```
[6](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L6)
</details>

### [L-29] State variables are not limited to a reasonable range

There are some missing limits in these functions, and this could lead to unexpected scenarios. Consider adding a min/max limit for the following values, when appropriate.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/RestakeManager.sol

216: maxDepositTVL = _maxDepositTVL;
```
[216](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L216)
</details>

### [L-30] Missing contract-existence checks before low-level calls

Low-level calls return success if there is no code present at the specified address, and this could lead to unexpected scenarios.

Ensure that the code is initialized by checking `<address>.code.length > 0`

<details>
<summary><i>4 issue instances in 4 files:</i></summary>

```solidity
File: contracts/TimelockController.sol

369: (bool success, ) = target.call{ value: value }(data);
```
[369](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L369)

```solidity
File: contracts/Rewards/RewardHandler.sol

68: (bool success, ) = rewardDestination.call{ value: balance }("");
```
[68](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L68)

```solidity
File: contracts/Deposits/DepositQueue.sol

168: (bool success, ) = feeAddress.call{ value: feeAmount }("");
```
[168](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L168)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

520: (bool success, ) = destination.call{ value: remainingAmount }("");
```
[520](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L520)
</details>

### [L-31] External calls in an unbounded loop may revert

Consider limiting the number of iterations in loops that make external calls, as just a single one of them failing will result in a revert.

<details>
<summary><i>20 issue instances in 5 files:</i></summary>

```solidity
File: contracts/RestakeManager.sol

/// @audit 2 external calls in unbounded for-loop -> 289: for (uint256 i = 0; i < odLength; ) {
307: operatorValues[j] = renzoOracle.lookupTokenValue(
317: totalWithdrawalQueueValue += renzoOracle.lookupTokenValue(
/// @audit 2 external calls in unbounded for-loop -> 299: for (uint256 j = 0; j < tokenLength; ) {
/// @audit 2 external calls in unbounded for-loop -> 683: for (uint256 i = 0; i < tokenLength; ) {
685: uint256 tokenRewardAmount = depositQueue.totalEarned(address(collateralTokens[i]));
688: totalRewards += renzoOracle.lookupTokenValue(collateralTokens[i], tokenRewardAmount);
```
[307](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L307) | [317](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L317) | [685](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L685) | [688](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L688)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

/// @audit 1 external calls in unbounded for-loop -> 176: for (uint256 i = 0; i < strategyLength; i++) {
177: if (strategyManager.stakerStrategyList(address(this), i) == _strategy) {
/// @audit 1 external calls in unbounded for-loop -> 209: for (uint256 i; i < tokens.length; ) {
212: queuedWithdrawalParams[0].strategies[i] = eigenPodManager.beaconChainETHStrategy();
/// @audit 4 external calls in unbounded for-loop -> 277: for (uint256 i; i < tokens.length; ) {
286: uint256 bufferToFill = withdrawQueue.getBufferDeficit(address(tokens[i]));
297: tokens[i].safeApprove(address(restakeManager.depositQueue()), bufferToFill);
300: restakeManager.depositQueue().fillERC20withdrawBuffer(
/// @audit 1 external calls in unbounded for-loop -> 381: for (uint256 i = 0; i < validatorFields.length; ) {
382: uint64 validatorCurrentBalanceGwei = BeaconChainProofs.getEffectiveBalanceGwei(
```
[177](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L177) | [212](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L212) | [286](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L286) | [297](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L297) | [300](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L300) | [382](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L382)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol

/// @audit 1 external calls in unbounded for-loop -> 167: for (uint256 _i; _i < _bridgesLength; ++_i) {
168: XERC20(_xerc20).setLimits(_bridges[_i], _minterLimits[_i], _burnerLimits[_i]);
```
[168](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L168)

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol

/// @audit 1 external calls in unbounded for-loop -> 109: for (uint256 _i; _i < _bridgesLength; ++_i) {
110: OptimismMintableXERC20(_xerc20).setLimits(
```
[110](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L110)

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol

/// @audit 8 external calls in unbounded for-loop -> 218: for (uint256 i = 0; i < _destinationParam.length; ) {
219: Client.EVM2AnyMessage memory evm2AnyMessage = Client.EVM2AnyMessage({
223: extraArgs: Client._argsToBytes(
225: Client.EVMExtraArgsV1({ gasLimit: 200_000 })
232: uint256 fees = linkRouterClient.getFee(
237: if (fees > linkToken.balanceOf(address(this)))
238: revert NotEnoughBalance(linkToken.balanceOf(address(this)), fees);
241: linkToken.approve(address(linkRouterClient), fees);
244: bytes32 messageId = linkRouterClient.ccipSend(
```
[219](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L219) | [223](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L223) | [225](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L225) | [232](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L232) | [237](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L237) | [238](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L238) | [241](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L241) | [244](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L244)
</details>

### [L-32] Input Arrays Lengths Not Checked

When a function takes two or more arrays as arguments, it is important to ensure that their lengths are equal to prevent unexpected behavior.
Add a check to compare the lengths of the arrays in the function body.

<details>
<summary><i>8 issue instances in 6 files:</i></summary>

```solidity
File: contracts/TimelockController.sol

215: function hashOperationBatch(
        address[] calldata targets,
        uint256[] calldata values,
        bytes[] calldata payloads,
        bytes32 predecessor,
        bytes32 salt
    ) public pure virtual returns (bytes32) {
```
[215](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L215)

```solidity
File: contracts/RestakeManager.sol

400: function chooseOperatorDelegatorForWithdraw(
        uint256 tokenIndex,
        uint256 ezETHValue,
        uint256[][] memory operatorDelegatorTokenTVLs,
        uint256[] memory operatorDelegatorTVLs,
        uint256 totalTVL
    ) public view returns (IOperatorDelegator) {
```
[400](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L400)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

364: function verifyWithdrawalCredentials(
        uint64 oracleTimestamp,
        BeaconChainProofs.StateRootProof calldata stateRootProof,
        uint40[] calldata validatorIndices,
        bytes[] calldata withdrawalCredentialProofs,
        bytes32[][] calldata validatorFields
    ) external onlyNativeEthRestakeAdmin {
405: function verifyAndProcessWithdrawals(
        uint64 oracleTimestamp,
        BeaconChainProofs.StateRootProof calldata stateRootProof,
        BeaconChainProofs.WithdrawalProof[] calldata withdrawalProofs,
        bytes[] calldata validatorFieldsProofs,
        bytes32[][] calldata validatorFields,
        bytes32[][] calldata withdrawalFields
    ) external onlyNativeEthRestakeAdmin {
446: function recoverTokens(
        IERC20[] memory tokenList,
        uint256[] memory amountsToWithdraw,
        address recipient
    ) external onlyNativeEthRestakeAdmin {
```
[364](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L364) | [405](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L405) | [446](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L446)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol

73: function deployXERC20(
        string memory _name,
        string memory _symbol,
        uint256[] memory _minterLimits,
        uint256[] memory _burnerLimits,
        address[] memory _bridges,
        address _proxyAdmin
    ) external returns (address _xerc20) {
```
[73](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L73)

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol

36: function deployOptimismMintableXERC20(
        string memory _name,
        string memory _symbol,
        uint256[] memory _minterLimits,
        uint256[] memory _burnerLimits,
        address[] memory _bridges,
        address _proxyAdmin,
        address _l1Token
    ) public returns (address _xerc20) {
```
[36](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L36)

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol

210: function sendPrice(
        CCIPDestinationParam[] calldata _destinationParam,
        ConnextDestinationParam[] calldata _connextDestinationParam
    ) external payable onlyPriceFeedSender nonReentrant {
```
[210](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L210)
</details>

### [L-33] Critical functions should be a two step procedure

Critical functions in Solidity contracts should follow a two-step procedure to enhance security, minimize human error, and ensure proper access control. By dividing sensitive operations into distinct phases, such as initiation and confirmation, developers can introduce a safeguard against unintended actions or unauthorized access.

<details>
<summary><i>11 issue instances in 6 files:</i></summary>

```solidity
File: contracts/Withdraw/WithdrawQueue.sol

106: function updateWithdrawBufferTarget(
        TokenWithdrawBuffer[] calldata _newBufferTarget
    ) external onlyWithdrawQueueAdmin {
129: function updateCoolDownPeriod(uint256 _newCoolDownPeriod) external onlyWithdrawQueueAdmin {
```
[106](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L106) | [129](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L129)

```solidity
File: contracts/Rewards/RewardHandler.sol

71: function setRewardDestination(
        address _rewardDestination
    ) external nonReentrant onlyRestakeManagerAdmin {
```
[71](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L71)

```solidity
File: contracts/RestakeManager.sol

187: function setOperatorDelegatorAllocation(
        IOperatorDelegator _operatorDelegator,
        uint256 _allocationBasisPoints
    ) external onlyRestakeManagerAdmin {
215: function setMaxDepositTVL(uint256 _maxDepositTVL) external onlyRestakeManagerAdmin {
708: function setTokenTvlLimit(IERC20 _token, uint256 _limit) external onlyRestakeManagerAdmin {
```
[187](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L187) | [215](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L215) | [708](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L708)

```solidity
File: contracts/Oracle/RenzoOracle.sol

54: function setOracleAddress(
        IERC20 _token,
        AggregatorV3Interface _oracleAddress
    ) external nonReentrant onlyOracleAdmin {
```
[54](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L54)

```solidity
File: contracts/Deposits/DepositQueue.sol

87: function setWithdrawQueue(IWithdrawQueue _withdrawQueue) external onlyRestakeManagerAdmin {
93: function setFeeConfig(
        address _feeAddress,
        uint256 _feeBasisPoints
    ) external onlyRestakeManagerAdmin {
112: function setRestakeManager(IRestakeManager _restakeManager) external onlyRestakeManagerAdmin {
```
[87](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L87) | [93](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L93) | [112](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L112)

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol

36: function setOracleAddress(AggregatorV3Interface _oracleAddress) external onlyOwner {
```
[36](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L36)
</details>

### [L-34] `onlyOwner` functions not accessible if `owner` renounces ownership

The `owner` is able to perform certain privileged activities, but it's possible to set the `owner` to `address(0)`. This can represent a certain risk if the ownership is renounced for any other reason than by design.

Renouncing ownership will leave the contract without an `owner`, therefore limiting any functionality that needs authority.

<details>
<summary><i>20 issue instances in 5 files:</i></summary>

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol

135: function setLimits(
        address _bridge,
        uint256 _mintingLimit,
        uint256 _burningLimit
    ) external onlyOwner {
```
[135](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L135)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

320: function updatePriceByOwner(uint256 price) external onlyOwner {
466: function setAllowedBridgeSweeper(address _sweeper, bool _allowed) external onlyOwner {
478: function recoverNative(uint256 _amount, address _to) external onlyOwner {
489: function recoverERC20(address _token, uint256 _amount, address _to) external onlyOwner {
501: function setOraclePriceFeed(IRenzoOracleL2 _oracle) external onlyOwner {
511: function setReceiverPriceFeed(address _receiver) external onlyOwner {
521: function updateBridgeFeeShare(uint256 _newShare) external onlyOwner {
532: function updateSweepBatchSize(uint256 _newBatchSize) external onlyOwner {
```
[320](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L320) | [466](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L466) | [478](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L478) | [489](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L489) | [501](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L501) | [511](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L511) | [521](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L521) | [532](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L532)

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol

92: function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {
103: function updateCCIPEthChainSelector(uint32 _newChainDomain) external onlyOwner {
113: function unPause() external onlyOwner {
121: function pause() external onlyOwner {
130: function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {
```
[92](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L92) | [103](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L103) | [113](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L113) | [121](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L121) | [130](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L130)

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol

96: function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {
107: function updateCCIPEthChainSelector(uint64 _newChainSelector) external onlyOwner {
117: function unPause() external onlyOwner {
125: function pause() external onlyOwner {
134: function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {
```
[96](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L96) | [107](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L107) | [117](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L117) | [125](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L125) | [134](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L134)

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol

36: function setOracleAddress(AggregatorV3Interface _oracleAddress) external onlyOwner {
```
[36](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L36)
</details>

### [L-35] safeApprove is deprecated

The usage of deprecated library functions should be discouraged, as safeApprove is also potentially [dangerous](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2219).

<details>
<summary><i>10 issue instances in 5 files:</i></summary>

```solidity
File: contracts/RestakeManager.sol

552: _collateralToken.safeApprove(address(depositQueue), bufferToFill);
559: _collateralToken.safeApprove(address(operatorDelegator), _amount);
664: _token.safeApprove(address(operatorDelegator), _amount);
```
[552](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L552) | [559](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L559) | [664](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L664)

```solidity
File: contracts/Deposits/DepositQueue.sol

142: IERC20(_asset).safeApprove(address(withdrawQueue), _amount);
```
[142](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L142)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

164: _token.safeApprove(address(strategyManager), _tokenAmount);
297: tokens[i].safeApprove(address(restakeManager.depositQueue()), bufferToFill);
```
[164](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L164) | [297](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L297)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

369: depositToken.safeApprove(address(connext), _amountIn);
429: collateralToken.safeApprove(address(connext), balance);
```
[369](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L369) | [429](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L429)

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol

84: SafeERC20.safeApprove(IERC20(_erc20), lockbox, _amount);
86: SafeERC20.safeApprove(IERC20(xerc20), blastStandardBridge, _amount);
```
[84](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L84) | [86](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L86)
</details>

### [L-36] Loss of precision on division

Solidity doesn't support fractions, so divisions by large numbers could result in the quotient being zero.

To avoid this, it's recommended to require a minimum numerator amount to ensure that it is always greater than the denominator.

<details>
<summary><i>20 issue instances in 6 files:</i></summary>

```solidity
File: contracts/RestakeManager.sol

379: (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
                    BASIS_POINTS /
380: BASIS_POINTS /
                    BASIS_POINTS
421: (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
                    BASIS_POINTS /
422: BASIS_POINTS /
                    BASIS_POINTS &&
```
[379](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L379) | [380](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L380) | [421](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L421) | [422](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L422)

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol

40: return (10 ** 18 * totalTVL) / totalSupply;
```
[40](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L40)

```solidity
File: contracts/Oracle/RenzoOracle.sol

80: return (uint256(price) * _balance) / SCALE_FACTOR;
97: return (_value * SCALE_FACTOR) / uint256(price);
135: uint256 inflationPercentaage = (SCALE_FACTOR * _newValueAdded) /
            (_currentValueInProtocol + _newValueAdded);
139: uint256 newEzETHSupply = (_existingEzETHSupply * SCALE_FACTOR) /
            (SCALE_FACTOR - inflationPercentaage);
158: uint256 redeemAmount = (_currentValueInProtocol * _ezETHBeingBurned) / _existingEzETHSupply;
```
[80](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L80) | [97](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L97) | [135](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L135) | [139](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L139) | [158](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L158)

```solidity
File: contracts/Deposits/DepositQueue.sol

167: feeAmount = (msg.value * feeBasisPoints) / 10000;
261: feeAmount = (balance * feeBasisPoints) / 10000;
```
[167](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L167) | [261](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L261)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol

241: bridges[_bridge].minterParams.ratePerSecond = _limit / _DURATION;
263: bridges[_bridge].burnerParams.ratePerSecond = _limit / _DURATION;
```
[241](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L241) | [263](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L263)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

253: uint256 xezETHAmount = (1e18 * amountOut) / _lastPrice;
280: return (_amountIn * bridgeFeeShare) / FEE_BASIS;
282: return (sweepBatchSize * bridgeFeeShare) / FEE_BASIS;
338: (_price > lastPrice && (_price - lastPrice) > (lastPrice / 10)) ||
339: (_price < lastPrice && (lastPrice - _price) > (lastPrice / 10))
386: uint256 fee = (amountNextWETH * bridgeRouterFeeBps) / 10_000;
```
[253](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L253) | [280](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L280) | [282](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L282) | [338](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L338) | [339](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L339) | [386](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L386)
</details>

### [L-37] Unbounded Gas Consumption on External Calls

Consider using `addr.call{gas: <amount>}("")` to set a gas limit and prevent potential reversion due to gas consumption.

<details>
<summary><i>5 issue instances in 5 files:</i></summary>

```solidity
File: contracts/TimelockController.sol

369: (bool success, ) = target.call{ value: value }(data);
```
[369](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L369)

```solidity
File: contracts/Rewards/RewardHandler.sol

68: (bool success, ) = rewardDestination.call{ value: balance }("");
```
[68](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L68)

```solidity
File: contracts/Deposits/DepositQueue.sol

168: (bool success, ) = feeAddress.call{ value: feeAmount }("");
```
[168](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L168)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

520: (bool success, ) = destination.call{ value: remainingAmount }("");
```
[520](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L520)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol

131: (bool _success, ) = payable(_to).call{ value: _amount }("");
```
[131](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L131)
</details>

### [L-38] Lack of two-step update for updating protocol addresses

Add a two-step process for any critical address changes.

<details>
<summary><i>13 issue instances in 7 files:</i></summary>

```solidity
File: contracts/Rewards/RewardHandler.sol

/// @audit `rewardDestination` is changed
76: rewardDestination = _rewardDestination;
```
[76](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L76)

```solidity
File: contracts/Deposits/DepositQueue.sol

/// @audit `withdrawQueue` is changed
90: withdrawQueue = _withdrawQueue;
/// @audit `feeAddress` is changed
104: feeAddress = _feeAddress;
/// @audit `feeBasisPoints` is changed
106: feeBasisPoints = _feeBasisPoints;
/// @audit `restakeManager` is changed
114: restakeManager = _restakeManager;
```
[90](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L90) | [104](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L104) | [106](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L106) | [114](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L114)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

/// @audit `tokenStrategyMapping[_token]` is changed
111: tokenStrategyMapping[_token] = _strategy;
/// @audit `delegateAddress` is changed
124: delegateAddress = _delegateAddress;
/// @audit `baseGasAmountSpent` is changed
137: baseGasAmountSpent = _baseGasAmountSpent;
```
[111](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L111) | [124](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L124) | [137](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L137)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

/// @audit `oracle` is changed
503: oracle = _oracle;
/// @audit `receiver` is changed
513: receiver = _receiver;
```
[503](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L503) | [513](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L513)

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol

/// @audit `xRenzoBridgeL1` is changed
95: xRenzoBridgeL1 = _newXRenzoBridgeL1;
```
[95](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L95)

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol

/// @audit `xRenzoBridgeL1` is changed
99: xRenzoBridgeL1 = _newXRenzoBridgeL1;
```
[99](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L99)

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol

/// @audit `oracle` is changed
43: oracle = _oracleAddress;
```
[43](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L43)
</details>

### [L-39] Some `ERC20` can revert on a zero value `transfer`

Not all `ERC20` implementations are totally compliant, and some (e.g `LEND`) may fail while transfering a zero amount.

<details>
<summary><i>1 issue instances in 1 files:</i></summary>

```solidity
File: contracts/RestakeManager.sol

/// @audit `_amount` has not been checked for zero value before transfer
540: _collateralToken.safeTransferFrom(msg.sender, address(this), _amount);
```
[540](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L540)
</details>

### [L-40] Functions calling contracts with transfer hooks are missing reentrancy guards

Even if the function follows the best practice of check-effects-interaction, not using a reentrancy guard when there may be transfer hooks will open the users of this protocol up to read-only reentrancies with no way to protect against it, except by block-listing the whole protocol.

<details>
<summary><i>9 issue instances in 6 files:</i></summary>

```solidity
File: contracts/RestakeManager.sol

/// @audit function `depositTokenRewardsFromProtocol()` 
661: _token.safeTransferFrom(msg.sender, address(this), _amount);
```
[661](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L661)

```solidity
File: contracts/Deposits/DepositQueue.sol

/// @audit function `sweepERC20()` 
262: IERC20(token).safeTransfer(feeAddress, feeAmount);
```
[262](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L262)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

/// @audit function `recoverNative()` 
479: payable(_to).transfer(_amount);
/// @audit function `recoverERC20()` 
490: IERC20(_token).safeTransfer(_to, _amount);
```
[479](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L479) | [490](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L490)

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol

/// @audit function `recoverNative()` 
295: payable(_to).transfer(_amount);
/// @audit function `recoverERC20()` 
306: IERC20(_token).safeTransfer(_to, _amount);
```
[295](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L295) | [306](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L306)

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol

/// @audit function `bridgeTo()` 
83: SafeERC20.safeTransferFrom(IERC20(_erc20), msg.sender, address(this), _amount);
```
[83](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L83)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol

/// @audit function `_withdraw()` 
134: ERC20.safeTransfer(_to, _amount);
/// @audit function `_deposit()` 
147: ERC20.safeTransferFrom(msg.sender, address(this), _amount);
```
[134](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L134) | [147](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L147)
</details>

### [L-41] Missing `__gap[50]` storage variable in Upgradeable contracts

In the context of upgradeable contracts, ensuring forward compatibility between different contract versions is a critical concern. One key strategy for ensuring this compatibility is the use of a `__gap` storage variable.

The `__gap` variable acts as a reserved space in the contract's storage layout. This space can then be utilized for adding new state variables in future contract upgrades, while maintaining the original storage layout of the contract.

Without the `__gap` storage variable, adding new state variables in a contract upgrade can risk overwriting the existing contract storage, potentially leading to unpredictable behavior or data loss.

Therefore, if your contract is designed to be upgradeable, it's crucial to include a `__gap` storage variable. The absence of this variable in an upgradeable contract can signify a potential risk for future upgrades.

<details>
<summary><i>13 issue instances in 13 files:</i></summary>

```solidity
File: contracts/token/EzEthToken.sol

13: contract EzEthToken is Initializable, ERC20Upgradeable, IEzEthToken, EzEthTokenStorageV1 {
```
[13](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L13)

```solidity
File: contracts/Withdraw/WithdrawQueue.sol

10: contract WithdrawQueue is
    Initializable,
    PausableUpgradeable,
    ReentrancyGuardUpgradeable,
    WithdrawQueueStorageV1
{
```
[10](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L10)

```solidity
File: contracts/Rewards/RewardHandler.sol

16: contract RewardHandler is Initializable, ReentrancyGuardUpgradeable, RewardHandlerStorageV1 {
```
[16](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L16)

```solidity
File: contracts/RestakeManager.sol

26: contract RestakeManager is Initializable, ReentrancyGuardUpgradeable, RestakeManagerStorageV2 {
```
[26](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L26)

```solidity
File: contracts/Permissions/RoleManager.sol

14: contract RoleManager is IRoleManager, AccessControlUpgradeable, RoleManagerStorageV3 {
```
[14](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L14)

```solidity
File: contracts/Oracle/RenzoOracle.sol

13: contract RenzoOracle is
    IRenzoOracle,
    Initializable,
    ReentrancyGuardUpgradeable,
    RenzoOracleStorageV1
{
```
[13](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L13)

```solidity
File: contracts/Deposits/DepositQueue.sol

9: contract DepositQueue is Initializable, ReentrancyGuardUpgradeable, DepositQueueStorageV2 {
```
[9](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L9)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

17: contract OperatorDelegator is
    Initializable,
    ReentrancyGuardUpgradeable,
    OperatorDelegatorStorageV4
{
```
[17](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L17)

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol

15: contract XERC20 is
    Initializable,
    ERC20Upgradeable,
    OwnableUpgradeable,
    IXERC20,
    ERC20PermitUpgradeable
{
```
[15](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L15)

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol

10: contract OptimismMintableXERC20 is ERC165Upgradeable, XERC20, IOptimismMintableERC20 {
```
[10](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L10)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

26: contract xRenzoDeposit is
    Initializable,
    OwnableUpgradeable,
    ReentrancyGuardUpgradeable,
    IRateProvider,
    xRenzoDepositStorageV3
{
```
[26](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L26)

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol

10: contract RenzoOracleL2 is Initializable, OwnableUpgradeable, RenzoOracleL2StorageV1 {
```
[10](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L10)

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol

15: contract xRenzoBridge is
    IXReceiver,
    Initializable,
    ReentrancyGuardUpgradeable,
    xRenzoBridgeStorageV1
{
```
[15](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L15)
</details>

### [L-42] Use of ownership with a single step rather than double

A single step ownership change is risky due to the fact that the new owner address could be wrong.

Instead, consider using a contract like [Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol), which provides a two-step ownership.

<details>
<summary><i>7 issue instances in 5 files:</i></summary>

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol

16: contract XERC20 is
    Initializable,
    ERC20Upgradeable,
    OwnableUpgradeable,
```
[16](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L16)

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol

27: contract xRenzoDeposit is
    Initializable,
    OwnableUpgradeable,
```
[27](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L27)

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol

5: import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol";
10: contract ConnextReceiver is IXReceiver, Ownable, Pausable {
```
[5](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L5) | [10](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L10)

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol

8: import { Ownable } from "@openzeppelin/contracts/access/Ownable.sol";
14: contract Receiver is CCIPReceiver, Ownable, Pausable {
```
[8](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L8) | [14](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L14)

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol

11: contract RenzoOracleL2 is Initializable, OwnableUpgradeable, RenzoOracleL2StorageV1 {
```
[11](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L11)
</details>

### [L-43] Consider using OpenZeppelin's SafeCast for any casting

OpenZeppelin's has `SafeCast` library that provides functions to safely cast. Recommended to use it instead of casting directly in any case.

<details>
<summary><i>8 issue instances in 5 files:</i></summary>

```solidity
File: contracts/Oracle/RenzoOracle.sol

80: uint256(price)
97: uint256(price)
```
[80](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L80) | [97](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L97)

```solidity
File: contracts/Oracle/Mantle/METHShim.sol

80: int256(methStaking.mETHToETH(1 ether))
```
[80](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L80)

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol

80: int256(wBETHToken.exchangeRate())
```
[80](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L80)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

207: uint96(delegationManager.cumulativeWithdrawalsQueued(address(this)))
343: uint256(-podOwnerShares)
344: uint256(podOwnerShares)
```
[207](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L207) | [343](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L343) | [344](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L344)

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol

54: uint256(price)
```
[54](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L54)
</details>

### [L-44] Using Low-Level Call for Transfers

Directly using low-level calls for Ether transfers can introduce vulnerabilities and obscure the intent of your transfers.
Adopt modern Solidity best practices by switching to recognized libraries like `SafeTransferLib.safeTransferETH` or `Address.sendValue`.
This ensures safer transactions, enhances code clarity, and aligns with the standards of the Solidity community.

<details>
<summary><i>4 issue instances in 3 files:</i></summary>

```solidity
File: contracts/Rewards/RewardHandler.sol

68: (bool success, ) = rewardDestination.call{ value: balance }("");
```
[68](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L68)

```solidity
File: contracts/Deposits/DepositQueue.sol

168: (bool success, ) = feeAddress.call{ value: feeAmount }("");
286: (bool success, ) = payable(msg.sender).call{ value: gasRefund }("");
```
[168](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L168) | [286](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L286)

```solidity
File: contracts/Delegation/OperatorDelegator.sol

520: (bool success, ) = destination.call{ value: remainingAmount }("");
```
[520](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L520)
</details>

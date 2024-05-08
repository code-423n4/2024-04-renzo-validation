## Low Findings

|        | Issue                                                                            |
| ------ | -------------------------------------------------------------------------------- |
| [L-01] | Deprecated OpenZeppelin functions should not be used                             |
| [L-02] | Unsafe ERC20 Operations should not be used                                       |
| [L-03] | Solidity pragma should be specific, not wide                                     |
| [L-04] | Missing checks for `address(0)` when assigning values to address state variables |
| [L-05] | The `nonReentrant` `modifier` should occur before all other modifiers            |
| [L-06] | Modifiers invoked only once can be shoe-horned into the function                 |
| [L-07] | Upgradable contracts not taken into account                                      |
| [L-08] | Consider checking that transfer to `address(this)` is disabled                   |
| [L-09] | Unchecked Return Values of the approve() Function                                |
| [L-10] | Unsafe use of `transfer()` with IERC20                                           |
| [L-11] | Unchecked Return Values of `transfer()`                                          |
| [L-12] | Owner can renounce Ownership                                                     |
| [L-13] | Missing Contract-Existence Checks Before Low-Level Calls                         |
| [L-14] | Consider implementing two-step procedure for updating protocol addresses         |
| [L-15] | Consider Using Ownable2Step rather than Ownable                                  |

Total: 15 issues

## NonCritical Findings

|        | Issue                                                                                      |
| ------ | ------------------------------------------------------------------------------------------ |
| [N-01] | Define and use `constant` variables instead of using literals                              |
| [N-02] | Large literal values multiples of 10000 can be replaced with scientific notation           |
| [N-03] | Internal functions called only once can be inlined                                         |
| [N-04] | Inconsistency in declaring uint256/uint (or) int256/int variables within a contract        |
| [N-05] | Unused Custom Errors                                                                       |
| [N-06] | Style guide: Contract does not follow the Solidity style guide's suggested layout ordering |
| [N-07] | Missing event for critical arithmetic parameters in `OperatorDelegator.stakeEth()`         |
| [N-08] | Costly operations inside a loop                                                            |
| [N-09] | Unused state variables                                                                     |
| [N-10] | Variable could be marked as immutable                                                      |
| [N-11] | Consider using a struct rather than having many function input parameters                  |
| [N-12] | Consider using descriptive constants when passing zero as a function argument              |
| [N-13] | Custom error has no error details                                                          |
| [N-14] | Consider using OpenZeppelin's SafeCast for any casting                                     |
| [N-15] | Consider emitting an event at the end of the constructor                                   |
| [N-16] | Consider using named returns                                                               |
| [N-17] | Events are missing sender information                                                      |
| [N-18] | Leverage Recent Solidity Features with 0.8.23                                              |
| [N-19] | High cyclomatic complexity                                                                 |
| [N-20] | Contract uses both `require()/revert()` as well as custom errors                           |
| [N-21] | Solidity Version Too Recent to be Trusted                                                  |
| [N-22] | Critical system parameter changes should be behind a timelock                              |
| [N-23] | Outdated Solidity Version                                                                  |
| [N-24] | Function/Constructor Argument Names Not in mixedCase                                       |
| [N-25] | Non-uppercase/Constant-case Naming for immutable Variables                                 |
| [N-26] | Consider defining system-wide constants in a single file                                   |
| [N-27] | Style guide: Control structures do not follow the Solidity Style Guide                     |
| [N-28] | Consider using named mappings                                                              |
| [N-29] | Contracts should have full test coverage                                                   |
| [N-30] | Large or complicated code bases should implement invariant tests                           |

Total: 30 issues

## Low Findings Details

[L-01] Deprecated OpenZeppelin functions should not be used
Openzeppelin has deprecated several functions and replaced with newer versions. Please refer to https://docs.openzeppelin.com/

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol [Line: 84](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L84)

    ```solidity
            SafeERC20.safeApprove(IERC20(_erc20), lockbox, _amount);
    ```

-   Found in contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol [Line: 86](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L86)

    ```solidity
            SafeERC20.safeApprove(IERC20(xerc20), blastStandardBridge, _amount);
    ```

-   Found in contracts/Bridge/L1/xRenzoBridge.sol [Line: 181](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L181)

    ```solidity
            ezETH.safeApprove(address(xezETHLockbox), ezETHAmount);
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 369](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L369)

    ```solidity
            depositToken.safeApprove(address(connext), _amountIn);
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 429](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L429)

    ```solidity
            collateralToken.safeApprove(address(connext), balance);
    ```

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 164](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L164)

    ```solidity
            _token.safeApprove(address(strategyManager), _tokenAmount);
    ```

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 297](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L297)

    ```solidity
                        tokens[i].safeApprove(address(restakeManager.depositQueue()), bufferToFill);
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 142](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L142)

    ```solidity
            IERC20(_asset).safeApprove(address(withdrawQueue), _amount);
    ```

-   Found in contracts/RestakeManager.sol [Line: 552](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L552)

    ```solidity
                _collateralToken.safeApprove(address(depositQueue), bufferToFill);
    ```

-   Found in contracts/RestakeManager.sol [Line: 559](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L559)

    ```solidity
            _collateralToken.safeApprove(address(operatorDelegator), _amount);
    ```

-   Found in contracts/RestakeManager.sol [Line: 664](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L664)

    ```solidity
            _token.safeApprove(address(operatorDelegator), _amount);
    ```

-   Found in contracts/TimelockController.sol [Line: 99](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L99)

    ```solidity
            _setupRole(TIMELOCK_ADMIN_ROLE, address(this));
    ```

-   Found in contracts/TimelockController.sol [Line: 103](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L103)

    ```solidity
                _setupRole(TIMELOCK_ADMIN_ROLE, admin);
    ```

-   Found in contracts/TimelockController.sol [Line: 108](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L108)

    ```solidity
                _setupRole(PROPOSER_ROLE, proposers[i]);
    ```

-   Found in contracts/TimelockController.sol [Line: 109](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L109)

    ```solidity
                _setupRole(CANCELLER_ROLE, proposers[i]);
    ```

-   Found in contracts/TimelockController.sol [Line: 114](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L114)

          ```solidity
                      _setupRole(EXECUTOR_ROLE, executors[i]);
          ```

</details>

### [L-02] Unsafe ERC20 Operations should not be used

ERC20 functions may not behave as expected. For example: return values are not always meaningful. It is recommended to use OpenZeppelin's SafeERC20 library.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/L1/xRenzoBridge.sol [Line: 241](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L241)

    ```solidity
                linkToken.approve(address(linkRouterClient), fees);
    ```

-   Found in contracts/Bridge/L1/xRenzoBridge.sol [Line: 295](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L295)

    ```solidity
            payable(_to).transfer(_amount);
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 479](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L479)

    ```solidity
            payable(_to).transfer(_amount);
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 268](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L268)

    ```solidity
                token.approve(address(restakeManager), balance - feeAmount);
    ```

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 303](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L303)

    ```solidity
                payable(msg.sender).transfer(_withdrawRequest.amountToRedeem);
    ```

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 305](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L305)

    ```solidity
                IERC20(_withdrawRequest.collateralToken).transfer(
    ```

</details>

### [L-03] Solidity pragma should be specific, not wide

Consider using a specific version of Solidity in your contracts instead of a wide version. For example, instead of `pragma solidity ^0.8.0;`, use `pragma solidity 0.8.18;`

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L2)

    ```solidity
    pragma solidity ^0.8.13;
    ```

-   Found in contracts/Bridge/Connext/libraries/LibConnextStorage.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/libraries/LibConnextStorage.sol#L2)

    ```solidity
    pragma solidity ^0.8.0;
    ```

-   Found in contracts/Bridge/Connext/libraries/TokenId.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/libraries/TokenId.sol#L2)

    ```solidity
    pragma solidity ^0.8.0;
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L2)

    ```solidity
    pragma solidity >=0.8.4 <0.9.0;
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20Factory.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L2)

    ```solidity
    pragma solidity >=0.8.4 <0.9.0;
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L2)

    ```solidity
    pragma solidity >=0.8.4 <0.9.0;
    ```

-   Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L2)

    ```solidity
    pragma solidity >=0.8.4 <0.9.0;
    ```

-   Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L2)

    ```solidity
    pragma solidity >=0.8.4 <0.9.0;
    ```

-   Found in contracts/Bridge/xERC20/interfaces/IOptimismMintableERC20.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/interfaces/IOptimismMintableERC20.sol#L2)

    ```solidity
    pragma solidity >=0.8.4 <0.9.0;
    ```

-   Found in contracts/Bridge/xERC20/interfaces/IXERC20.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/interfaces/IXERC20.sol#L2)

    ```solidity
    pragma solidity >=0.8.4 <0.9.0;
    ```

-   Found in contracts/Bridge/xERC20/interfaces/IXERC20Factory.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/interfaces/IXERC20Factory.sol#L2)

    ```solidity
    pragma solidity >=0.8.4 <0.9.0;
    ```

-   Found in contracts/Bridge/xERC20/interfaces/IXERC20Lockbox.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/interfaces/IXERC20Lockbox.sol#L2)

    ```solidity
    pragma solidity >=0.8.4 <0.9.0;
    ```

-   Found in contracts/EigenLayer/interfaces/IAVSDirectory.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IAVSDirectory.sol#L2)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/interfaces/IBeaconChainOracle.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IBeaconChainOracle.sol#L2)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/interfaces/IDelayedWithdrawalRouter.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IDelayedWithdrawalRouter.sol#L2)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/interfaces/IDelegationFaucet.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IDelegationFaucet.sol#L2)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/interfaces/IDelegationManager.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IDelegationManager.sol#L2)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/interfaces/IETHPOSDeposit.sol [Line: 12](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IETHPOSDeposit.sol#L12)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/interfaces/IEigenPod.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IEigenPod.sol#L2)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/interfaces/IEigenPodManager.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IEigenPodManager.sol#L2)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/interfaces/IPausable.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IPausable.sol#L2)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/interfaces/IPauserRegistry.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IPauserRegistry.sol#L2)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/interfaces/ISignatureUtils.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/ISignatureUtils.sol#L2)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/interfaces/ISlasher.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/ISlasher.sol#L2)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/interfaces/IStrategy.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IStrategy.sol#L2)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/interfaces/IStrategyManager.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IStrategyManager.sol#L2)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/interfaces/IWhitelister.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IWhitelister.sol#L2)

    ```solidity
    pragma solidity >=0.5.0;
    ```

-   Found in contracts/EigenLayer/libraries/BytesLib.sol [Line: 9](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/BytesLib.sol#L9)

    ```solidity
    pragma solidity >=0.8.0 <0.9.0;
    ```

-   Found in contracts/EigenLayer/libraries/Endian.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Endian.sol#L2)

    ```solidity
    pragma solidity ^0.8.0;
    ```

-   Found in contracts/EigenLayer/libraries/Merkle.sol [Line: 4](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Merkle.sol#L4)

    ```solidity
    pragma solidity ^0.8.0;
    ```

-   Found in contracts/TimelockController.sol [Line: 4](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L4)

    ```solidity
    pragma solidity ^0.8.0;
    ```

-   Found in contracts/token/EzEthToken.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L2)

    ```solidity
    pragma solidity ^0.8.9;
    ```

</details>

### [L-04] Missing checks for `address(0)` when assigning values to address state variables

State variables that are of type address should always be checked to ensure that they are not being assigned the null address (address(0x0)). A null address often implies an error or omission in the code. Without proper checks, such a scenario can lead to unexpected behavior and potential vulnerabilities in the smart contract.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 138](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L138)

    ```solidity
            receiver = _receiver;
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 149](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L149)

    ```solidity
            oracle = _oracle;
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 467](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L467)

    ```solidity
            allowedBridgeSweepers[_sweeper] = _allowed;
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 503](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L503)

    ```solidity
            oracle = _oracle;
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 513](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L513)

    ```solidity
            receiver = _receiver;
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 86](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L86)

    ```solidity
            FACTORY = _factory;
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 123](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L123)

    ```solidity
            lockbox = _lockbox;
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 208](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L208)

    ```solidity
            bridges[_bridge].minterParams.currentLimit = _currentLimit - _change;
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 220](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L220)

    ```solidity
            bridges[_bridge].burnerParams.currentLimit = _currentLimit - _change;
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 233](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L233)

    ```solidity
            bridges[_bridge].minterParams.maxLimit = _limit;
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 235](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L235)

    ```solidity
            bridges[_bridge].minterParams.currentLimit = _calculateNewCurrentLimit(
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 241](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L241)

    ```solidity
            bridges[_bridge].minterParams.ratePerSecond = _limit / _DURATION;
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 255](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L255)

    ```solidity
            bridges[_bridge].burnerParams.maxLimit = _limit;
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 257](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L257)

    ```solidity
            bridges[_bridge].burnerParams.currentLimit = _calculateNewCurrentLimit(
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 263](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L263)

    ```solidity
            bridges[_bridge].burnerParams.ratePerSecond = _limit / _DURATION;
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20Factory.sol [Line: 58](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L58)

    ```solidity
            lockboxImplementation = _lockboxImplementation;
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20Factory.sol [Line: 59](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L59)

    ```solidity
            xerc20Implementation = _xerc20Implementation;
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 45](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L45)

    ```solidity
            XERC20 = IXERC20(_xerc20);
    ```

-   Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 46](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L46)

    ```solidity
            ERC20 = IERC20(_erc20);
    ```

-   Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 44](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L44)

    ```solidity
            l1Token = _l1Token;
    ```

-   Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 45](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L45)

    ```solidity
            optimismBridge = _optimismBridge;
    ```

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 112](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L112)

    ```solidity
            tokenStrategyMapping[_token] = _strategy;
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 272](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L272)

    ```solidity
                totalEarned[address(token)] = totalEarned[address(token)] + balance - feeAmount;
    ```

-   Found in contracts/RestakeManager.sol [Line: 111](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L111)

    ```solidity
            roleManager = _roleManager;
    ```

-   Found in contracts/RestakeManager.sol [Line: 112](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L112)

    ```solidity
            ezETH = _ezETH;
    ```

-   Found in contracts/RestakeManager.sol [Line: 113](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L113)

    ```solidity
            renzoOracle = _renzoOracle;
    ```

-   Found in contracts/RestakeManager.sol [Line: 114](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L114)

    ```solidity
            strategyManager = _strategyManager;
    ```

-   Found in contracts/RestakeManager.sol [Line: 115](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L115)

    ```solidity
            delegationManager = _delegationManager;
    ```

-   Found in contracts/RestakeManager.sol [Line: 116](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L116)

    ```solidity
            depositQueue = _depositQueue;
    ```

-   Found in contracts/RestakeManager.sol [Line: 154](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L154)

    ```solidity
            operatorDelegatorAllocations[_newOperatorDelegator] = _allocationBasisPoints;
    ```

-   Found in contracts/RestakeManager.sol [Line: 209](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L209)

    ```solidity
            operatorDelegatorAllocations[_operatorDelegator] = _allocationBasisPoints;
    ```

-   Found in contracts/RestakeManager.sol [Line: 714](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L714)

        ```solidity
                collateralTokenTvlLimits[_token] = _limit;
        ```

    </details>

### [L-05] | The `nonReentrant` `modifier` should occur before all other modifiers

This is the recommended way to protect against reentrancy in other modifiers.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/L1/xRenzoBridge.sol [Line: 213](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L213)

    ```solidity
        ) external payable onlyPriceFeedSender nonReentrant {
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 216](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L216)

    ```solidity
        ) external onlyNativeEthRestakeAdmin nonReentrant {
    ```

</details>

### [L-06] Modifiers invoked only once can be shoe-horned into the function

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/L1/xRenzoBridge.sol [Line: 55](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L55)

    ```solidity
        modifier onlyPriceFeedSender() {
    ```

-   Found in contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol [Line: 43](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L43)

    ```solidity
        modifier onlySource(address _originSender, uint32 _origin) {
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 62](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L62)

    ```solidity
        modifier onlyERC20RewardsAdmin() {
    ```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 24](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L24)

    ```solidity
        modifier onlyOracleAdmin() {
    ```

-   Found in contracts/RestakeManager.sol [Line: 77](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L77)

    ```solidity
        modifier onlyDepositWithdrawPauserAdmin() {
    ```

-   Found in contracts/Rewards/RewardHandler.sol [Line: 18](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L18)

    ```solidity
        modifier onlyNativeEthRestakeAdmin() {
    ```

-   Found in contracts/Rewards/RewardHandler.sol [Line: 24](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L24)

    ```solidity
        modifier onlyRestakeManagerAdmin() {
    ```

-   Found in contracts/token/EzEthToken.sol [Line: 21](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L21)

    ```solidity
        modifier onlyTokenAdmin() {
    ```

</details>

### [L-07] Upgradable contracts not taken into account

In the realm of blockchain development, it's crucial to consider the impact of upgradable contracts, especially when handling token addresses through interfaces like IERC20. These contracts can evolve over time, potentially altering their behavior or interface. Such changes may lead to compatibility issues or security vulnerabilities in the protocol that relies on them.

### [L-08] Consider checking that transfer to `address(this)` is disabled

Functions allowing users to specify the receiving address should check whether the referenced address is not `address(this)`. This would prevent tokens to remain lock inside the contract due to an incorrect transfer or mint.

### [L-09] Unchecked Return Values of the approve() Function

The unchecked return value of the `approve()` method can potentially cause transaction failures to go unnoticed in your contract.

Some IERC20 token implementations utilize boolean return values to indicate transaction failures, instead of relying on the `revert()` function. If the return value of the `approve()` method isn't appropriately verified, transactions may seemingly proceed even when the necessary token approvals have not been appropriately executed.

As a safeguard, it is crucial to consistently verify the return values of these methods. This helps to ensure the correct execution of token approvals, maintaining the integrity of transaction operations and reducing the risk of unnoticed transaction failures.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Deposits/DepositQueue.sol [Line: 268](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol#L268C13-L268C27)

```solidity
token.approve(address(restakeManager), balance - feeAmount);
```

-   Found in contracts/Bridge/L1/xRenzoBridge.sol [Line: 241](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L241)

```solidity
linkToken.approve(address(linkRouterClient), fees);
```

</details>

### [L-10] | Unsafe use of `transfer()` with IERC20

As a safer alternative, consider using OpenZeppelin's SafeERC20 library, which offers `safeTransfer()` and `safeTransferFrom()` functions to address these incompatibilities.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 305](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L305)

```solidity
IERC20(_withdrawRequest.collateralToken).transfer(
                msg.sender,
                _withdrawRequest.amountToRedeem
            );
```

</details>

### [L-11] Unchecked Return Values of `transfer()`

`transfer()` or `transferFrom()` methods without checking their return values. Some IERC20 implementations use these boolean return values to signal transaction failures instead of using `revert()`. Not checking these values could allow transactions to proceed without the intended token transfers. It is recommended to always check the return values of these methods.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 305](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L305)

```solidity
IERC20(_withdrawRequest.collateralToken).transfer(
                msg.sender,
                _withdrawRequest.amountToRedeem
            );
```

</details>

### [L-12] Owner can renounce Ownership

Some of the contracts utilize OpenZeppelin's Ownable(Upgradable).sol and contains `onlyOwner` functions. Consider implementing custom logic for `renounceOwnership()` to handle renouncing ownership.

### [L-13] Missing Contract-Existence Checks Before Low-Level Calls

When making low-level calls, it's crucial to ensure the existence of the contract at the specified address. If the contract doesn't exist at the given address, low-level calls will still return success, potentially causing errors in the code execution. Therefore, alongside zero-address checks, adding an additional check to verify that

.code.length > 0 before making low-level calls would be recommended.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/XERC20Lockbox.sol [Line: 131](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L131)

```solidity
(bool _success, ) = payable(_to).call{ value: _amount }("");
```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 168](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol#L168)

```solidity
(bool success, ) = feeAddress.call{ value: feeAmount }("");
```

-   Found in contracts/Rewards/RewardHandler.sol [Line: 68](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Rewards/RewardHandler.sol#L68)

```solidity
(bool success, ) = rewardDestination.call{ value: balance }("");
```

</details>

### [L-14] Consider implementing two-step procedure for updating protocol addresses

Direct state address changes in a function can be risky, as they don't allow for a verification step before the change is made. It's safer to implement a two-step process where the new address is first proposed, then later confirmed, allowing for more control and the chance to catch errors or malicious activity.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol [Line: 130](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L130)

```solidity
  function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {
        if (_newXRenzoDeposit == address(0)) revert InvalidZeroInput();
        emit XRenzoDepositUpdated(_newXRenzoDeposit, address(xRenzoDeposit));
        xRenzoDeposit = IxRenzoDeposit(_newXRenzoDeposit);
    }
```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 87](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol#L87)

```solidity
    function setWithdrawQueue(IWithdrawQueue _withdrawQueue) external onlyRestakeManagerAdmin {
        if (address(_withdrawQueue) == address(0)) revert InvalidZeroInput();
        emit WithdrawQueueUpdated(address(withdrawQueue), address(_withdrawQueue));
        withdrawQueue = _withdrawQueue;
    }
```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 112](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol#L112)

```solidity
    function setRestakeManager(IRestakeManager _restakeManager) external onlyRestakeManagerAdmin {
        if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();

        restakeManager = _restakeManager;

        emit RestakeManagerUpdated(_restakeManager);
    }
```

</details>

### [L-15] | Consider Using Ownable2Step rather than Ownable

For more robust security and prevent inadvertent ownership transfers, it's advisable to use Ownable2Step or Ownable2StepUpgradeable. Contracts necessitate an active confirmation from the recipient before the ownership transfer is finalized. This mechanism serves as a safeguard against scenarios where, for instance, a typo in the address could lead to unintentional ownership changes.

By implementing a two-step confirmation process, contracts can ensure the accurate and intentional transfer of ownership.

## Non-Critical Findings Details

### [N-01] Define and use `constant` variables instead of using literals

Magic numbers are hardcoded numerical values used directly in the code, making it harder to read and maintain. Consider using constants instead of magic numbers.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/L2/Oracle/RenzoOracleL2.sol [Line: 28](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L28)

    ```solidity
            if (_oracle.decimals() > 18) revert InvalidTokenDecimals(18, _oracle.decimals());
    ```

-   Found in contracts/Bridge/L2/Oracle/RenzoOracleL2.sol [Line: 37](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L37)

    ```solidity
            if (_oracleAddress.decimals() > 18) {
    ```

-   Found in contracts/Bridge/L2/Oracle/RenzoOracleL2.sol [Line: 38](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L38)

    ```solidity
                revert InvalidTokenDecimals(18, _oracleAddress.decimals());
    ```

-   Found in contracts/Bridge/L2/Oracle/RenzoOracleL2.sol [Line: 53](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L53)

    ```solidity
            uint256 _scaledPrice = (uint256(price)) * 10 ** (18 - oracle.decimals());
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 140](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L140)

    ```solidity
            bridgeRouterFeeBps = 5;
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 152](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L152)

    ```solidity
            bridgeFeeShare = 5;
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 155](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L155)

    ```solidity
            sweepBatchSize = 32 ether;
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 338](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L338)

    ```solidity
                (_price > lastPrice && (_price - lastPrice) > (lastPrice / 10)) ||
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 339](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L339)

    ```solidity
                (_price < lastPrice && (lastPrice - _price) > (lastPrice / 10))
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 533](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L533)

    ```solidity
            if (_newBatchSize < 32 ether) revert InvalidSweepBatchSize(_newBatchSize);
    ```

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 81](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L81)

    ```solidity
            if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 82](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L82)

    ```solidity
            if (address(_strategyManager) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 83](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L83)

    ```solidity
            if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 84](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L84)

    ```solidity
            if (address(_delegationManager) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 85](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L85)

    ```solidity
            if (address(_eigenPodManager) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 110](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L110)

    ```solidity
            if (address(_token) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 122](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L122)

    ```solidity
            if (address(_delegateAddress) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 123](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L123)

    ```solidity
            if (address(delegateAddress) != address(0x0)) revert DelegateAddressAlreadySet();
    ```

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 147](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L147)

    ```solidity
            if (address(tokenStrategyMapping[token]) == address(0x0) || tokenAmount == 0)
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 77](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L77)

    ```solidity
            if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 99](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L99)

    ```solidity
                if (_feeAddress == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 103](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L103)

    ```solidity
            if (_feeBasisPoints > 10000) revert OverMaxBasisPoints();
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 113](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L113)

    ```solidity
            if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 166](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L166)

    ```solidity
            if (feeAddress != address(0x0) && feeBasisPoints > 0) {
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 167](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L167)

    ```solidity
                feeAmount = (msg.value * feeBasisPoints) / 10000;
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 171](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L171)

    ```solidity
                emit ProtocolFeesPaid(IERC20(address(0x0)), feeAmount, feeAddress);
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 179](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L179)

    ```solidity
            totalEarned[address(0x0)] = totalEarned[address(0x0)] + remainingRewards;
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 182](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L182)

    ```solidity
            emit RewardsDeposited(IERC20(address(0x0)), remainingRewards);
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 202](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L202)

    ```solidity
            emit ETHStakedFromQueue(operatorDelegator, pubkey, 32 ether, address(this).balance);
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 239](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L239)

    ```solidity
                    32 ether,
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 260](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L260)

    ```solidity
                if (feeAddress != address(0x0) && feeBasisPoints > 0) {
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 261](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L261)

    ```solidity
                    feeAmount = (balance * feeBasisPoints) / 10000;
    ```

-   Found in contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 129](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/BeaconChainProofs.sol#L129)

    ```solidity
                    32 * ((VALIDATOR_TREE_HEIGHT + 1) + BEACON_STATE_FIELD_TREE_HEIGHT),
    ```

-   Found in contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 162](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/BeaconChainProofs.sol#L162)

    ```solidity
                stateRootProof.length == 32 * (BEACON_BLOCK_HEADER_FIELD_TREE_HEIGHT),
    ```

-   Found in contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 214](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/BeaconChainProofs.sol#L214)

    ```solidity
                    32 * (executionPayloadHeaderFieldTreeHeight + WITHDRAWALS_TREE_HEIGHT + 1),
    ```

-   Found in contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 219](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/BeaconChainProofs.sol#L219)

    ```solidity
                    32 * (BEACON_BLOCK_HEADER_FIELD_TREE_HEIGHT + BEACON_BLOCK_BODY_FIELD_TREE_HEIGHT),
    ```

-   Found in contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 223](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/BeaconChainProofs.sol#L223)

    ```solidity
                withdrawalProof.slotProof.length == 32 * (BEACON_BLOCK_HEADER_FIELD_TREE_HEIGHT),
    ```

-   Found in contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 227](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/BeaconChainProofs.sol#L227)

    ```solidity
                withdrawalProof.timestampProof.length == 32 * (executionPayloadHeaderFieldTreeHeight),
    ```

-   Found in contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 233](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/BeaconChainProofs.sol#L233)

    ```solidity
                    32 *
    ```

-   Found in contracts/EigenLayer/libraries/BytesLib.sol [Line: 370](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/BytesLib.sol#L370)

    ```solidity
            require(_bytes.length >= _start + 32, "toUint256_outOfBounds");
    ```

-   Found in contracts/EigenLayer/libraries/BytesLib.sol [Line: 381](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/BytesLib.sol#L381)

    ```solidity
            require(_bytes.length >= _start + 32, "toBytes32_outOfBounds");
    ```

-   Found in contracts/EigenLayer/libraries/Endian.sol [Line: 16](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Endian.sol#L16)

    ```solidity
                (n >> 56) |
    ```

-   Found in contracts/EigenLayer/libraries/Endian.sol [Line: 17](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Endian.sol#L17)

    ```solidity
                ((0x00FF000000000000 & n) >> 40) |
    ```

-   Found in contracts/EigenLayer/libraries/Endian.sol [Line: 18](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Endian.sol#L18)

    ```solidity
                ((0x0000FF0000000000 & n) >> 24) |
    ```

-   Found in contracts/EigenLayer/libraries/Endian.sol [Line: 19](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Endian.sol#L19)

    ```solidity
                ((0x000000FF00000000 & n) >> 8) |
    ```

-   Found in contracts/EigenLayer/libraries/Endian.sol [Line: 20](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Endian.sol#L20)

    ```solidity
                ((0x00000000FF000000 & n) << 8) |
    ```

-   Found in contracts/EigenLayer/libraries/Endian.sol [Line: 21](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Endian.sol#L21)

    ```solidity
                ((0x0000000000FF0000 & n) << 24) |
    ```

-   Found in contracts/EigenLayer/libraries/Endian.sol [Line: 22](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Endian.sol#L22)

    ```solidity
                ((0x000000000000FF00 & n) << 40) |
    ```

-   Found in contracts/EigenLayer/libraries/Endian.sol [Line: 23](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Endian.sol#L23)

    ```solidity
                ((0x00000000000000FF & n) << 56);
    ```

-   Found in contracts/EigenLayer/libraries/Merkle.sol [Line: 54](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Merkle.sol#L54)

    ```solidity
                proof.length != 0 && proof.length % 32 == 0,
    ```

-   Found in contracts/EigenLayer/libraries/Merkle.sol [Line: 58](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Merkle.sol#L58)

    ```solidity
            for (uint256 i = 32; i <= proof.length; i += 32) {
    ```

-   Found in contracts/EigenLayer/libraries/Merkle.sol [Line: 113](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Merkle.sol#L113)

    ```solidity
                proof.length != 0 && proof.length % 32 == 0,
    ```

-   Found in contracts/EigenLayer/libraries/Merkle.sol [Line: 117](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Merkle.sol#L117)

    ```solidity
            for (uint256 i = 32; i <= proof.length; i += 32) {
    ```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 40](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L40)

    ```solidity
            if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 54](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L54)

    ```solidity
            if (address(_token) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 57](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L57)

    ```solidity
            if (_oracleAddress.decimals() != 18) {
    ```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 58](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L58)

    ```solidity
                revert InvalidTokenDecimals(18, _oracleAddress.decimals());
    ```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 70](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L70)

    ```solidity
            if (address(oracle) == address(0x0)) revert OracleNotFound();
    ```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 84](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L84)

    ```solidity
            if (address(oracle) == address(0x0)) revert OracleNotFound();
    ```

-   Found in contracts/RateProvider/BalancerRateProvider.sol [Line: 21](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L21)

    ```solidity
            if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/RateProvider/BalancerRateProvider.sol [Line: 22](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L22)

    ```solidity
            if (address(_ezETHToken) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/RestakeManager.sol [Line: 146](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L146)

    ```solidity
            if (_allocationBasisPoints > (100 * BASIS_POINTS)) revert OverMaxBasisPoints();
    ```

-   Found in contracts/RestakeManager.sol [Line: 191](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L191)

    ```solidity
            if (address(_operatorDelegator) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/RestakeManager.sol [Line: 192](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L192)

    ```solidity
            if (_allocationBasisPoints > (100 * BASIS_POINTS)) revert OverMaxBasisPoints();
    ```

-   Found in contracts/RestakeManager.sol [Line: 231](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L231)

    ```solidity
            if (IERC20Metadata(address(_newCollateralToken)).decimals() != 18)
    ```

-   Found in contracts/RestakeManager.sol [Line: 233](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L233)

    ```solidity
                    18,
    ```

-   Found in contracts/RestakeManager.sol [Line: 615](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L615)

    ```solidity
            emit Deposit(msg.sender, IERC20(address(0x0)), msg.value, ezETHToMint, _referralId);
    ```

-   Found in contracts/RestakeManager.sol [Line: 679](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L679)

    ```solidity
            totalRewards += depositQueue.totalEarned(address(0x0));
    ```

-   Found in contracts/Rewards/RewardHandler.sol [Line: 41](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L41)

    ```solidity
            if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/Rewards/RewardHandler.sol [Line: 42](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L42)

    ```solidity
            if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();
    ```

-   Found in contracts/Rewards/RewardHandler.sol [Line: 75](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L75)

        ```solidity
                if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();
        ```

    </details>

### [N-02] Large literal values multiples of 10000 can be replaced with scientific notation

Use `e` notation, for example: `1e18`, instead of its full numeric value.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/L1/xRenzoBridge.sol [Line: 225](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L225)

    ```solidity
                        Client.EVMExtraArgsV1({ gasLimit: 200_000 })
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 40](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L40)

    ```solidity
        uint32 public constant FEE_BASIS = 10000;
    ```

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 386](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L386)

    ```solidity
                uint256 fee = (amountNextWETH * bridgeRouterFeeBps) / 10_000;
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 103](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L103)

    ```solidity
            if (_feeBasisPoints > 10000) revert OverMaxBasisPoints();
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 167](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L167)

    ```solidity
                feeAmount = (msg.value * feeBasisPoints) / 10000;
    ```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 261](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L261)

    ```solidity
                    feeAmount = (balance * feeBasisPoints) / 10000;
    ```

</details>

### [N-03] | Internal functions called only once can be inlined

Instead of separating the logic into a separate function, consider inlining the logic into the calling function. This can reduce the number of function calls and increase readability.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 339](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/BeaconChainProofs.sol#L339)

    ```solidity
        function getWithdrawalTimestamp(
    ```

-   Found in contracts/EigenLayer/libraries/Merkle.sol [Line: 48](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Merkle.sol#L48)

    ```solidity
        function processInclusionProofKeccak(
    ```

-   Found in contracts/EigenLayer/libraries/Merkle.sol [Line: 107](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/Merkle.sol#L107)

    ```solidity
        function processInclusionProofSha256(
    ```

-   Found in contracts/EigenLayer/libraries/StructuredLinkedList.sol [Line: 161](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/StructuredLinkedList.sol#L161)

    ```solidity
        function remove(List storage self, uint256 _node) internal returns (uint256) {
    ```

</details>

### [N-04] | Inconsistency in declaring uint256/uint (or) int256/int variables within a contract

Consider keeping the naming convention consistent in a given contract

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 17](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L17)

    ```solidity
    contract OperatorDelegator is
    ```

</details>

### [N-05] | Unused Custom Errors

It is recommended that the definition be removed when custom error is unused

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Errors/Errors.sol [Line: 32](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Errors/Errors.sol#L32)

    ```solidity
    error WithdrawAlreadyCompleted();
    ```

-   Found in contracts/Errors/Errors.sol [Line: 35](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Errors/Errors.sol#L35)

    ```solidity
    error NotOriginalWithdrawCaller(address expectedCaller);
    ```

-   Found in contracts/Errors/Errors.sol [Line: 98](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Errors/Errors.sol#L98)

    ```solidity
    error InvalidOrigin();
    ```

</details>

### [N-06] Style guide: Contract does not follow the Solidity style guide's suggested layout ordering

Adhering to a recommended order in Solidity contracts increases code readability and maintenance.
[More information in Documentation](https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout)
It's recommended to use the following order:

1. Type declarations
2. State variables
3. Events
4. Errors
5. Modifiers
6. Functions

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/L1/xRenzoBridge.sol [Line: 61](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L61)

    ```solidity
        uint8 public constant EXPECTED_DECIMALS = 18;
    ```

-   Found in contracts/Bridge/xERC20/interfaces/IXERC20.sol [Line: 46](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/interfaces/IXERC20.sol#L46)

    ```solidity
        struct Bridge {
    ```

-   Found in contracts/EigenLayer/interfaces/IAVSDirectory.sol [Line: 17](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IAVSDirectory.sol#L17)

    ```solidity
        event AVSMetadataURIUpdated(address indexed avs, string metadataURI);
    ```

-   Found in contracts/EigenLayer/interfaces/IEigenPod.sol [Line: 34](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IEigenPod.sol#L34)

    ```solidity
        struct ValidatorInfo {
    ```

-   Found in contracts/EigenLayer/interfaces/IEigenPod.sol [Line: 63](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IEigenPod.sol#L63)

    ```solidity
        event EigenPodStaked(bytes pubkey);
    ```

-   Found in contracts/EigenLayer/interfaces/IStrategyManager.sol [Line: 26](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IStrategyManager.sol#L26)

    ```solidity
        event Deposit(address staker, IERC20 token, IStrategy strategy, uint256 shares);
    ```

-   Found in contracts/EigenLayer/interfaces/IStrategyManager.sol [Line: 157](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/interfaces/IStrategyManager.sol#L157)

    ```solidity
        struct DeprecatedStruct_WithdrawerAndNonce {
    ```

-   Found in contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 84](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/BeaconChainProofs.sol#L84)

    ```solidity
        struct WithdrawalProof {
    ```

-   Found in contracts/EigenLayer/libraries/StructuredLinkedList.sol [Line: 18](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/EigenLayer/libraries/StructuredLinkedList.sol#L18)

    ```solidity
        struct List {
    ```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 30](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L30)

    ```solidity
        event OracleAddressUpdated(IERC20 token, AggregatorV3Interface oracleAddress);
    ```

-   Found in contracts/RestakeManager.sol [Line: 38](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L38)

    ```solidity
        uint256 constant BASIS_POINTS = 100;
    ```

-   Found in contracts/RestakeManagerStorage.sol [Line: 29](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L29)

    ```solidity
        struct PendingWithdrawal {
    ```

-   Found in contracts/Rewards/RewardHandler.sol [Line: 29](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L29)

    ```solidity
        event RewardDestinationUpdated(address rewardDestination);
    ```

-   Found in contracts/TimelockController.sol [Line: 127](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L127)

    ```solidity
        modifier onlyRoleOrOpenRole(bytes32 role) {
    ```

-   Found in contracts/Withdraw/WithdrawQueueStorage.sol [Line: 12](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueueStorage.sol#L12)

    ```solidity
        struct TokenWithdrawBuffer {
    ```

</details>

### [N-07] | Missing event for critical arithmetic parameters in `OperatorDelegator.stakeEth()`

`OperatorDelegator#stakeEth()` does not emit an event, so it is difficult to track changes off-chain. Emit an event for critical parameter changes.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 358](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol#L358)

```solidity
      stakedButNotVerifiedEth += msg.value
```

</details>

### [N-08] | Costly operations inside a loop

Costly operations inside a loop might waste gas, so optimizations are recommended. Reading from state in a loop incurs a lot of gas because of expensive SLOADs, which might lead to an out-of-gas. Use a local variable to hold the loop computation result.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 385](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol#L385)

```solidity
stakedButNotVerifiedEth -= (validatorCurrentBalanceGwei * GWEI_TO_WEI);
```

</details>

### [N-09] | Unused state variables

By deleting these variables, it is possible to optimize the use of gas.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 79](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/BeaconChainProofs.sol#L79)

```solidity
    uint64 internal constant SECONDS_PER_EPOCH = SLOTS_PER_EPOCH * SECONDS_PER_SLOT;
```

-   Found in contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 81](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/BeaconChainProofs.sol#L81)

```solidity
    bytes8 internal constant UINT64_MASK = 0xffffffffffffffff;
```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 20](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol#L20)

```solidity
    string constant INVALID_0_INPUT = "Invalid 0 input";
```

</details>

### [N-10] | Variable could be marked as immutable

State variables that are not updated following deployment should be declared immutable to save gas.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol [Line: 12](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L12)

```solidity
address public connext;
```

</details>

### [N-11] Consider using a struct rather than having many function input parameters

Functions with many parameters can become difficult to read and maintain. Using a struct to encapsulate these parameters can increase code readability, increase reusability, and reduce the likelihood of errors. Consider refactoring functions that take more than three parameters to use a struct instead.

### [N-12] | Consider using descriptive constants when passing zero as a function argument

In instances where utilizing a zero parameter is essential, it is recommended to employ descriptive constants or an enum instead of directly integrating zero within function calls. This strategy aids in clearly articulating the caller's intention and minimizes the risk of errors. Emitting zero also not recomended, as it is not clear what the intention is.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/xERC20/contracts/XERC20Factory.sol [Line: 163](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L163)

```solidity
 _xerc20 = CREATE3.deploy(_salt, _bytecode, 0);
```

</details>

### [N-13] Custom error has no error details

Take advantage of custom error's return value property.

An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the () sign, providing a serious advantage in debugging and examining the revert details.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol [Line: 36](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L36)
-   Found in contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol [Line: 37](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L37)

```solidity
   error AmountLessThanZero();
   error InvalidAddress();
```

-   Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol [Line: 15](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L15)

```solidity
error OptimismMintableXERC20Factory_NoBridges();
```

-   Found in contracts/Errors/Errors.sol [Line: 5-26](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Errors/Errors.sol#L5-L26)

```solidity
error InvalidZeroInput();

error AlreadyAdded();

error NotFound();

error MaxTVLReached();

error NotRestakeManagerAdmin();

error NotDepositQueue();

error ContractPaused();

error OverMaxBasisPoints();
```

-   Found in contracts/Errors/Errors.sol [Line: 38-86](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Errors/Errors.sol#L38-L86)

```solidity
error NotOperatorDelegatorAdmin();

error NotOracleAdmin();

error NotRestakeManager();

error NotNativeEthRestakeAdmin();

error DelegateAddressAlreadySet();

error NotERC20RewardsAdmin();

error TransferFailed();

error NotEzETHMinterBurner();

error NotTokenAdmin();

error OracleNotFound();

error OraclePriceExpired();

error MismatchedArrayLengths();

error NotDepositWithdrawPauser();

error MaxTokenTVLReached();

error InvalidOraclePrice();

error NotImplemented();

error InvalidTokenAmount();
```

-   Found in contracts/Errors/Errors.sol [Line: 113-125](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Errors/Errors.sol#L113-L125)

```solidity
error UnauthorizedBridgeSweeper();

error NotBridgeAdmin();

error NotPriceFeedSender();

error UnAuthorisedCall();

error PriceFeedNotAvailable();
```

-   Found in contracts/Errors/Errors.sol [Line: 134-146](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Errors/Errors.sol#L134-L146)

```solidity
error NotWithdrawQueueAdmin();

error NotEnoughWithdrawBuffer();

error EarlyClaim();

error UnsupportedWithdrawAsset();

error InvalidWithdrawIndex();
```

</details>

### [N-14] Consider using OpenZeppelin's SafeCast for any casting

OpenZeppelin's has SafeCast library that provides functions to safely cast. Recommended to use it instead of casting directly in any case.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 80](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol#L80)

```solidity
return (uint256(price) * _balance) / SCALE_FACTOR;
```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 97](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol#L97)

```solidity
return (_value * SCALE_FACTOR) / uint256(price);
```

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 343-344](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol#L343-L344)

```solidity
podOwnerShares < 0
                ? queuedShares[IS_NATIVE] + stakedButNotVerifiedEth - uint256(-podOwnerShares)
                : queuedShares[IS_NATIVE] + stakedButNotVerifiedEth + uint256(podOwnerShares);
```

-   Found in contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 133](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/BeaconChainProofs.sol#L133)

```solidity
uint256(validatorIndex);
```

</details>

### [N-15] Consider emitting an event at the end of the constructor

This will allow users to easily exactly pinpoint when and by whom a contract was constructed.

<details>
<summary><i>Instances</i></summary>

Found in contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol [Line: 39](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L39)

```solidity
  constructor(address _blastStandardBridge, address _registry) {
        // Sanity check
        if (_blastStandardBridge == address(0) || _registry == address(0)) {
            revert InvalidAddress();
        }

        blastStandardBridge = _blastStandardBridge;
        registry = _registry;
    }
```

-   Found in contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol [Line: 52](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L52)

```solidity
constructor(address _connext, address _xRenzoBridgeL1, uint32 _connextEthChainDomain) {
        if (_xRenzoBridgeL1 == address(0) || _connextEthChainDomain == 0 || _connext == address(0))
            revert InvalidZeroInput();

        // Set connext bridge address
        connext = _connext;

        // Set xRenzoBridge L1 contract address
        xRenzoBridgeL1 = _xRenzoBridgeL1;

        // Set connext source chain Domain Id for Ethereum L1
        connextEthChainDomain = _connextEthChainDomain;

        // Pause The contract to setup xRenzoDeposit
        _pause();
    }
```

</details>

### [N-16] Consider using named returns

Using named returns makes the code more self-documenting, makes it easier to fill out NatSpec, and in some cases can save gas. The cases below are where there currently is at most one return statement, which is optimal for named returns.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Oracle/Binance/WBETHShim.sol [Line: 42](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/Binance/WBETHShim.sol#L42)

```solidity
function getRoundData(uint80) external pure returns (uint80, int256, uint256, uint256, uint80) {
```

</details>

### [N-17] | Events are missing sender information

Events should include the sender information when emitted in public or external functions for easier traceability.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/xERC20/contracts/XERC20Factory.sol [Line: 91](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L91)

```solidity
emit XERC20Deployed(_xerc20);
```

-   Found in contracts/Bridge/L1/xRenzoBridge.sol [Line: 196](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L196)

```solidity
emit EzETHMinted(_transferId, _amount, _origin, _originSender, ezETHAmount);
```

</details>

### [N-18] Leverage Recent Solidity Features with 0.8.23

The recent updates in Solidity provide several features and optimizations that, when leveraged appropriately, can significantly elevate your contract's code clarity and maintainability. Key things include the use of push0 for placing 0 on the stack for EVM versions starting from "Shanghai", making your code simpler and more straightforward. Moreover, Solidity has extended NatSpec documentation support to enum and struct definitions, facilitating more comprehensive and insightful code documentation.

Additionally, the re-implementation of the UnusedAssignEliminator and UnusedStoreEliminator in the Solidity optimizer provides the ability to remove unused assignments in deeply nested loops. This results in a cleaner, more efficient contract code, reducing clutter and potential points of confusion during code review or debugging. It's recommended to make full use of these features and optimizations to increase the robustness and readability of your smart contracts.

### [N-19] High cyclomatic complexity

Functions with high cyclomatic complexity are harder to understand, test, and maintain. Consider breaking down these blocks into more manageable units, by splitting things into utility functions, by reducing nesting, and by using early returns.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/xERC20/contracts/XERC20Factory.sol [Line: 210](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L210)

```solidity
function sendPrice(
        CCIPDestinationParam[] calldata _destinationParam,
        ConnextDestinationParam[] calldata _connextDestinationParam
    ) external payable onlyPriceFeedSender nonReentrant {
        // call getRate() to get the current price of ezETH
        uint256 exchangeRate = rateProvider.getRate();
        bytes memory _callData = abi.encode(exchangeRate, block.timestamp);
        // send price feed to renzo CCIP receivers
        for (uint256 i = 0; i < _destinationParam.length; ) {
            Client.EVM2AnyMessage memory evm2AnyMessage = Client.EVM2AnyMessage({
                receiver: abi.encode(_destinationParam[i]._renzoReceiver), // ABI-encoded xRenzoDepsot contract address
                data: _callData, // ABI-encoded ezETH exchange rate with Timestamp
                tokenAmounts: new Client.EVMTokenAmount[](0), // Empty array indicating no tokens are being sent
                extraArgs: Client._argsToBytes(
                    // Additional arguments, setting gas limit
                    Client.EVMExtraArgsV1({ gasLimit: 200_000 })
                ),
                // Set the feeToken  address, indicating LINK will be used for fees
                feeToken: address(linkToken)
            });

            // Get the fee required to send the message
            uint256 fees = linkRouterClient.getFee(
                _destinationParam[i].destinationChainSelector,
                evm2AnyMessage
            );

            if (fees > linkToken.balanceOf(address(this)))
                revert NotEnoughBalance(linkToken.balanceOf(address(this)), fees);

            // approve the Router to transfer LINK tokens on contract's behalf. It will spend the fees in LINK
            linkToken.approve(address(linkRouterClient), fees);

            // Send the message through the router and store the returned message ID
            bytes32 messageId = linkRouterClient.ccipSend(
                _destinationParam[i].destinationChainSelector,
                evm2AnyMessage
            );

            // Emit an event with message details
            emit MessageSent(
                messageId,
                _destinationParam[i].destinationChainSelector,
                _destinationParam[i]._renzoReceiver,
                exchangeRate,
                address(linkToken),
                fees
            );
            unchecked {
                ++i;
            }
        }

        // send price feed to renzo connext receiver
        for (uint256 i = 0; i < _connextDestinationParam.length; ) {
            connext.xcall{ value: _connextDestinationParam[i].relayerFee }(
                _connextDestinationParam[i].destinationDomainId,
                _connextDestinationParam[i]._renzoReceiver,
                address(0),
                msg.sender,
                0,
                0,
                _callData
            );

            emit ConnextMessageSent(
                _connextDestinationParam[i].destinationDomainId,
                _connextDestinationParam[i]._renzoReceiver,
                exchangeRate,
                _connextDestinationParam[i].relayerFee
            );

            unchecked {
                ++i;
            }
        }
    }
```

</details>

### [N-20] Contract uses both `require()/revert()` as well as custom errors

The contract uses both `require()/revert()` and custom errors for error handling. It's recommended to use a consistent approach for error handling in a single file.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/EigenLayer/libraries/Merkle.sol [Line: 53](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Merkle.sol#L53)

```solidity
 require(
            proof.length != 0 && proof.length % 32 == 0,
            "Merkle.processInclusionProofKeccak: proof length should be a non-zero multiple of 32"
        );
```

-   Found in contracts/EigenLayer/libraries/Merkle.sol [Line: 124](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Merkle.sol#L124)

```solidity
revert(0, 0)
```

-   Found in contracts/EigenLayer/libraries/Merkle.sol [Line: 134](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Merkle.sol#L134)

```solidity
revert(0, 0)
```

</details>

### [N-21] Solidity Version Too Recent to be Trusted

In several places in the protocol Solidity version used in the code is too recent to be trusted. Using a newer version might introduce unrecovered bugs. It is recommended to use version 0.8.19.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/xERC20/contracts/XERC20Factory.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L2)

```solidity
pragma solidity >=0.8.4 <0.9.0;
```

-   Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol)

```solidity
pragma solidity >=0.8.4 <0.9.0;
```

-   Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol)

```solidity
pragma solidity >=0.8.4 <0.9.0;
```

-   Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol)

```solidity
pragma solidity >=0.8.4 <0.9.0;
```

-   Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol)

```solidity
pragma solidity >=0.8.4 <0.9.0;
```

-   Found in contracts/Bridge/xERC20/interfaces/IOptimismMintableERC20.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IOptimismMintableERC20.sol)

```solidity
pragma solidity >=0.8.4 <0.9.0;
```

-   Found in contracts/Bridge/xERC20/interfaces/IXERC20Lockbox.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IXERC20Lockbox.sol)

```solidity
pragma solidity >=0.8.4 <0.9.0;
```

-   Found in contracts/Bridge/xERC20/interfaces/IXERC20Factory.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IXERC20Factory.sol)

```solidity
pragma solidity >=0.8.4 <0.9.0;
```

-   Found in contracts/Bridge/xERC20/interfaces/IXERC20.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IXERC20.sol)

```solidity
pragma solidity >=0.8.4 <0.9.0;
```

</details>

### [N-22] Critical system parameter changes should be behind a timelock

It is a recommended to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

<details>
<summary><i>Instances</i></summary>

Found in contracts/Deposits/DepositQueue.sol [Line: 93](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol#L93)

```solidity
function setFeeConfig(
        address _feeAddress,
        uint256 _feeBasisPoints
    ) external onlyRestakeManagerAdmin {
        if (_feeBasisPoints > 0) {
            if (_feeAddress == address(0x0)) revert InvalidZeroInput();
        }

        if (_feeBasisPoints > 10000) revert OverMaxBasisPoints();

        feeAddress = _feeAddress;
        feeBasisPoints = _feeBasisPoints;

        emit FeeConfigUpdated(_feeAddress, _feeBasisPoints);
    }
```

</details>

### [N-23] Outdated Solidity Version

The current Solidity version used in the contract is outdated. Consider using a more recent version for advanced features and security.

0.8.4: bytes.concat() instead of abi.encodePacked(,)

0.8.12: string.concat() instead of abi.encodePacked(,)

0.8.13: Ability to use using for with a list of free functions

0.8.14: ABI Encoder: When ABI-encoding values from calldata that contain nested arrays, correctly validate the nested array length against calldatasize() in all cases. Override Checker: Allow changing data location for parameters only when overriding external functions.

0.8.15: Code Generation: Avoid writing dirty bytes to storage when copying bytes arrays. Yul Optimizer: Keep all memory side-effects of inline assembly blocks.

0.8.16: Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.

0.8.17: Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.

<details>
<summary><i>Instances</i></summary>
-   Found in contracts/Bridge/Connext/libraries/TokenId.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/libraries/TokenId.sol#L2)

```solidity
pragma solidity ^0.8.0;
```

-   Found in contracts/Bridge/Connext/libraries/LibConnextStorage.sol [Line: 2](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/libraries/LibConnextStorage.sol#L2)

```solidity
pragma solidity ^0.8.0;
```

</details>

### [N-24] Function/Constructor Argument Names Not in mixedCase

Underscore before of after function argument names is a common convention in Solidity NOT a documentation requirement.

Function arguments should use mixedCase for readability and consistency with Solidity style guidelines.
Examples of recommended practice include: initialSupply, account, recipientAddress, senderAddress, newOwner.
[More information in Documentation](https://docs.soliditylang.org/en/v0.8.20/style-guide.html#function-argument-names)

Rule exceptions

-   Allow constant variable name/symbol/decimals to be lowercase (ERC20).
-   Allow `_` at the beginning of the mixedCase match for `private variables` and `unused parameters`.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 54](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L54)

```solidity
deposit(uint256 _amount)
```

-   Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 79](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L79)

```solidity
depositTo(address _to, uint256 _amount)
```

-   Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 91](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L91)

```solidity
depositNativeTo(address _to)
```

-   Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 100](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L100)

```solidity
withdraw(uint256 _amount)
```

-   Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 114](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L114)

```solidity
withdrawTo(address _to, uint256 _amount)
```

-   Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 104](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L104)

```solidity
_withdraw(address _to, uint256 _amount)
```

-   Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 57](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L57)

```solidity
_deposit(address _to, uint256 _amount)
```

-   Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 97](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol#L97)

```solidity
_mintWithCaller(address _caller, address _user, uint256 _amount)
```

-   Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol [Line: 37](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L37)

```solidity
deployOptimismMintableXERC20(
        string memory _name,
        string memory _symbol,
        uint256[] memory _minterLimits,
        uint256[] memory _burnerLimits,
        address[] memory _bridges,
        address _proxyAdmin,
        address _l1Token
    )
```

-   Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 48](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L48)

```solidity
supportsInterface(
        bytes4 interfaceId
    )
```

-   Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 64](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L64)

```solidity
mint(address _to, uint256 _amount)
```

-   Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 68](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L68)

```solidity
burn(address _from, uint256 _amount)
```

-   Found in contracts/Bridge/xERC20/interfaces/IOptimismMintableERC20.sol [Line: 15](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IOptimismMintableERC20.sol#L15)

```solidity
mint(address _to, uint256 _amount)
```

-   Found in contracts/Bridge/xERC20/interfaces/IOptimismMintableERC20.sol [Line: 17](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IOptimismMintableERC20.sol#L17)

```solidity
burn(address _from, uint256 _amount)
```

-   Found in contracts/Bridge/xERC20/interfaces/IXERC20.sol [Line: 98](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IXERC20.sol#L98)

```solidity
burningMaxLimitOf(address _bridge)
```

-   Found in contracts/Bridge/xERC20/interfaces/IXERC20.sol [Line: 107](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IXERC20.sol#L107)

```solidity
mintingCurrentLimitOf(address _minter)
```

-   Found in contracts/Bridge/xERC20/interfaces/IXERC20.sol [Line: 116](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IXERC20.sol#L116)

```solidity
burningCurrentLimitOf(address _bridge)
```

-   Found in contracts/Bridge/xERC20/interfaces/IXERC20.sol [Line: 16](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IXERC20.sol#L16)

```solidity
mint(address _user, uint256 _amount)
```

-   Found in contracts/Bridge/xERC20/interfaces/IXERC20.sol [Line: 17](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IXERC20.sol#L17)

```solidity
burn(address _user, uint256 _amount)
```

-   Found in contracts/Bridge/L1/IxRenzoBridge.sol [Line: 29](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/IxRenzoBridge.sol#L29)

```solidity
sendPrice(
        CCIPDestinationParam[] calldata _destinationParam,
        ConnextDestinationParam[] calldata _connextDestinationParam
    )
```

-   Found in contracts/Bridge/L1/xRenzoBridge.sol [Line: 139](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L139)

```solidity
xReceive(
        bytes32 _transferId,
        uint256 _amount,
        address _asset,
        address _originSender,
        uint32 _origin,
        bytes memory
    )
```

-   Found in contracts/Bridge/L1/xRenzoBridge.sol [Line: 210](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L210)

```solidity
sendPrice(
        CCIPDestinationParam[] calldata _destinationParam,
        ConnextDestinationParam[] calldata _connextDestinationParam
    )
```

-   Found in contracts/Bridge/L1/xRenzoBridge.sol [Line: 294](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L294)

```solidity
recoverNative(uint256 _amount, address _to)
```

-   Found in contracts/Bridge/L1/xRenzoBridge.sol [Line: 305](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L305)

```solidity
recoverERC20(address _token, uint256 _amount, address _to)
```

-   Found in contracts/Bridge/Connext/core/IXReceiver.sol [Line: 5](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IXReceiver.sol#L5)

```solidity
xReceive(
        bytes32 _transferId,
        uint256 _amount,
        address _asset,
        address _originSender,
        uint32 _origin,
        bytes memory _callData
    )
```

-   Found in contracts/Bridge/Connext/core/IWeth.sol [Line: 7](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IWeth.sol#L7)

```solidity
withdraw(uint256 value)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 14](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L14)

```solidity
xcall(
        uint32 _destination,
        address _to,
        address _asset,
        address _delegate,
        uint256 _amount,
        uint256 _slippage,
        bytes calldata _callData
    )
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 14](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L14)

```solidity
xcall(
        uint32 _destination,
        address _to,
        address _asset,
        address _delegate,
        uint256 _amount,
        uint256 _slippage,
        bytes calldata _callData,
        uint256 _relayerFee
    )
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 35](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L35)

```solidity
xcallIntoLocal(
        uint32 _destination,
        address _to,
        address _asset,
        address _delegate,
        uint256 _amount,
        uint256 _slippage,
        bytes calldata _callData
    )
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 45](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L45)

```solidity
execute(ExecuteArgs calldata _args)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 47](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L47)

```solidity
forceUpdateSlippage(TransferInfo calldata _params, uint256 _slippage)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 49](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L49)

```solidity
forceReceiveLocal(TransferInfo calldata _params)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 51](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L51)

```solidity
bumpTransfer(bytes32 _transferId)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 53](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L53)

```solidity
routedTransfers(bytes32 _transferId)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 55](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L55)

```solidity
transferStatus(bytes32 _transferId)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 57](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L57)

```solidity
remote(uint32 _domain)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 63](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L63)

```solidity
approvedSequencers(address _sequencer)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 73](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L73)

```solidity
getRouterApproval(address _router)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 75](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L75)

```solidity
getRouterRecipient(address _router)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 77](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L77)

```solidity
getRouterOwner(address _router)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 79](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L79)

```solidity
getProposedRouterOwner(address _router)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 81](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L81)

```solidity
getProposedRouterOwnerTimestamp(address _router)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 85](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L85)

```solidity
routerBalances(address _router, address _asset)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 87](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L87)

```solidity
getRouterApprovalForPortal(address _router)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 89](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L89)

```solidity
initializeRouter(address _owner, address _recipient)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 91](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L91)

```solidity
setRouterRecipient(address _router, address _recipient)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 93](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L93)

```solidity
proposeRouterOwner(address _router, address _proposed)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 95](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L95)

```solidity
acceptProposedRouterOwner(address _router)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 97](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L97)

```solidity
addRouterLiquidityFor(
        uint256 _amount,
        address _local,
        address _router
    )
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 97](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L97)

```solidity
addRouterLiquidity(uint256 _amount, address _local)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 105](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L105)

```solidity
removeRouterLiquidityFor(
        TokenId memory _canonical,
        uint256 _amount,
        address payable _to,
        address _router
    )
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 105](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L105)

```solidity
removeRouterLiquidity(
        TokenId memory _canonical,
        uint256 _amount,
        address payable _to
    )
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 119](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L119)

```solidity
adoptedToCanonical(address _adopted)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 121](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L121)

```solidity
approvedAssets(TokenId calldata _canonical)
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 126](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L126)

```solidity
swapExact(
        bytes32 canonicalId,
        uint256 amountIn,
        address assetIn,
        address assetOut,
        uint256 minAmountOut,
        uint256 deadline
    )
```

-   Found in contracts/Bridge/Connext/core/IConnext.sol [Line: 136](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol#L136)

```solidity
calculateSwap(
        bytes32 canonicalId,
        uint8 tokenIndexFrom,
        uint8 tokenIndexTo,
        uint256 dx
    )
```

-   Found in contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol [Line: 10](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L10)

```solidity
getXERC20(address erc20)
```

-   Found in contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol [Line: 12](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L12)

```solidity
getERC20(address xerc20)
```

-   Found in contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol [Line: 14](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L14)

```solidity
getLockbox(address erc20)
```

-   Found in contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol [Line: 18](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L18)

```solidity
bridgeERC20To(
        address _localToken,
        address _remoteToken,
        address _to,
        uint256 _amount,
        uint32 _minGasLimit,
        bytes calldata _extraData
    )
```

-   Found in contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol [Line: 56](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L56)

```solidity
bridgeTo(
        address _to,
        address _erc20,
        address _remoteToken,
        uint256 _amount,
        uint32 _minGasLimit,
        bytes calldata _extraData
    )
```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 32](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol#L32)

```solidity
initialize(IRoleManager _roleManager)
```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 49](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol#L49)

```solidity
setOracleAddress(IERC20 _token, AggregatorV3Interface _oracleAddress)
```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 68](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol#L68)

```solidity
lookupTokenValue(IERC20 _token, uint256 _balance)
```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 82](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol#L82)

```solidity
lookupTokenAmountFromValue(IERC20 _token, uint256 _value)
```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 97](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol#L97)

```solidity
lookupTokenValues(IERC20[] memory _tokens, uint256[] memory _balances)
```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 114](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol#L114)

```solidity
calculateMintAmount(uint256 _currentValueInProtocol, uint256 _newValueAdded, uint256 _existingEzETHSupply)
```

-   Found in contracts/Oracle/RenzoOracle.sol [Line: 141](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol#L141)

```solidity
calculateRedeemAmount(
        uint256 _ezETHBeingBurned,
        uint256 _existingEzETHSupply,
        uint256 _currentValueInProtocol
    )
```

-   Found in contracts/Oracle/IRenzoOracle.sol [Line: 7](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/IRenzoOracle.sol#L7)

```solidity
lookupTokenValue(IERC20 _token, uint256 _balance)
```

-   Found in contracts/Oracle/IRenzoOracle.sol [Line: 8](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/IRenzoOracle.sol#L8)

```solidity
lookupTokenAmountFromValue(
        IERC20 _token,
        uint256 _value
    )
```

</details>

### [N-25] | Non-uppercase/Constant-case Naming for immutable Variables

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol [Line: 31](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L31)

```solidity
immutable blastStandardBridge
```

-   Found in contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol [Line: 32](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L32)

```solidity
immutable registry
```

</details>

### [N-26] Consider defining system-wide constants in a single file

System-wide constants should be declared in a single file for easier maintainability and readability. This contract seems to contain constants which could potentially be system-wide and could be easier managed if they were centralized in a single location.

<details>
<summary><i>Instances</i></summary>

System-wide used constant 'EXPECTED_DECIMALS':

-   Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 37](https://github.com/your_username/your_repository/blob/master/contracts/Bridge/L2/xRenzoDeposit.sol#L37)
-   Found in contracts/Bridge/L1/xRenzoBridge.sol [Line: 61](https://github.com/your_username/your_repository/blob/master/contracts/Bridge/L1/xRenzoBridge.sol#L61)

System-wide used constant 'MAX_TIME_WINDOW':

-   Found in contracts/Bridge/L2/Oracle/RenzoOracleL2.sol [Line: 11](https://github.com/your_username/your_repository/blob/master/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L11)
-   Found in contracts/Oracle/RenzoOracle.sol [Line: 21](https://github.com/your_username/your_repository/blob/master/contracts/Oracle/RenzoOracle.sol#L21)

System-wide used constant 'IS_NATIVE':

-   Found in contracts/Delegation/OperatorDelegator.sol [Line: 26](https://github.com/your_username/your_repository/blob/master/contracts/Delegation/OperatorDelegator.sol#L26)
-   Found in contracts/Deposits/DepositQueue.sol [Line: 13](https://github.com/your_username/your_repository/blob/master/contracts/Deposits/DepositQueue.sol#L13)
-   Found in contracts/Withdraw/WithdrawQueueStorage.sol [Line: 10](https://github.com/your_username/your_repository/blob/master/contracts/Withdraw/WithdrawQueueStorage.sol#L10)

</details>

### [N-27] Style guide: Control structures do not follow the Solidity Style Guide

Following recommended practices for control structures in solidity code is vital for readability and maintainability. The control structures in contracts, libraries, functions, and structs should adhere to the following standards:

-   Braces denoting the body should open on the same line as the declaration and close on their own line at the same indentation level as the beginning of the declaration.
-   A single space should precede the opening brace.
-   Control structures such as 'if', 'else', 'while', and 'for' should also follow these spacing and brace placement recommendations.

It is advised to revisit the [control structures](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures) sections in documentation to ensure conformity with these recommended practices, fostering cleaner and more maintainable code.

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/Deposits/DepositQueue.sol [Line: 99](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol#L99)

```solidity
if (_feeAddress == address(0x0))
  revert InvalidZeroInput();
```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 103](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol#L103)

```solidity
if (_feeBasisPoints > 10000)
  revert OverMaxBasisPoints();
```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 113](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol#L113)

```solidity
if (address(_restakeManager) == address(0x0))
  revert InvalidZeroInput();
```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 138](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol#L138)

```solidity
if (_amount == 0 || _asset == address(0))
  revert InvalidZeroInput();
```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 169](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol#L169)

```solidity
if (!success)
  revert TransferFailed();
```

-   Found in contracts/Deposits/DepositQueue.sol [Line: 287](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol#L287)

```solidity
if (!success)
  revert TransferFailed();
```

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 40](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L40)

```solidity
if (!roleManager.isWithdrawQueueAdmin(msg.sender))
  revert NotWithdrawQueueAdmin();
```

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 46](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L46)

```solidity
if (msg.sender != address(restakeManager))
  revert NotRestakeManager();
```

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 51](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L51)

```solidity
if (msg.sender != address(restakeManager.depositQueue()))
  revert NotDepositQueue();
```

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 109](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L109)

```solidity
if (_newBufferTarget.length == 0)
  revert InvalidZeroInput();
```

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 111](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L111)

```solidity
if (_newBufferTarget[i].asset == address(0) || _newBufferTarget[i].bufferAmount == 0)
  revert InvalidZeroInput();
```

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 130](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L130)

```solidity
if (_newCoolDownPeriod == 0)
  revert InvalidZeroInput();
```

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 196](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L196)

```solidity
if (_asset == address(0) || _amount == 0)
  revert InvalidZeroInput();
```

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 208](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L208)

```solidity
if (_amount == 0 || _assetOut == address(0))
  revert InvalidZeroInput();
```

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 211](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L211)

```solidity
if (withdrawalBufferTarget[_assetOut] == 0)
  revert UnsupportedWithdrawAsset();
```

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 236](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L236)

```solidity
if (amountToRedeem > getAvailableToWithdraw(_assetOut))
  revert NotEnoughWithdrawBuffer();
```

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 281](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L281)

```solidity
if (withdrawRequestIndex >= withdrawRequests[msg.sender].length)
  revert InvalidWithdrawIndex();
```

-   Found in contracts/Withdraw/WithdrawQueue.sol [Line: 287](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol#L287)

```solidity
if (block.timestamp - _withdrawRequest.createdAt < coolDownPeriod)
  revert EarlyClaim();
```

</details>

### [N-28] Consider using named mappings

As of Solidity version 0.8.18, it is possible to use named mappings to clarify the purpose of each mapping in the code.
It is recommended to use this feature for code readability and maintainability.

More information: [Solidity 0.8.18 Release Announcement](https://blog.soliditylang.org/2023/02/01/solidity-0.8.18-release-announcement/)

<details>
<summary><i>Instances</i></summary>

-   Found in contracts/TimelockController.sol [Line: 32](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol#L32)

```solidity
mapping(bytes32 => uint256) private _timestamps;
```

-   Found in contracts/EigenLayer/libraries/StructuredLinkedList.sol [Line: 20](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/StructuredLinkedList.sol#L20)

```solidity
mapping(uint256 => mapping(bool => uint256)) list;
```

-   Found in contracts/Bridge/xERC20/contracts/XERC20Factory.sol [Line: 20](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L20)

```solidity
mapping(address => address) internal _lockboxRegistry;
```

-   Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 41](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol#L41)

```solidity
mapping(address => Bridge) public bridges;
```

</details>

### [N-29] | Contracts should have full test coverage

It's recommended to have full test coverage for all contracts. While 100% code coverage does not guarantee absence of bugs, it can catch simple bugs and reduce regressions during code modifications. Moreover, to achieve full coverage, authors often have to refactor their code into more modular components, each testable separately. This leads to lower interdependencies, and results in code that is easier to understand and audit.

### [N-30] Large or complicated code bases should implement invariant tests

Large or complex code bases should include invariant fuzzing tests, such as those provided by Echidna. These tests require the identification of invariants that should not be violated under any circumstances, with the fuzzer testing various inputs and function calls to ensure these invariants always hold. This is especially important for code with a lot of inline-assembly, complicated math, or complex interactions between contracts. Despite having 100% code coverage, bugs can still occur due to the order of operations a user performs. Extensive invariant tests can significantly reduce this testing gap.
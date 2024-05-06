Tittle:
INCREASE OPTIMIZER RUNS

Description:
Optimizer variable can optimize costs by increasing the number of runs
to something like 2000 at least in the config. This value is a tuning
parameter for deploy v/s runtime costs. Higher values optimize for lower
runtime cost.

hardhat.config.ts

    const config: any = {
      solidity: {
        version: "0.8.20",
        settings: {
          optimizer: {
            enabled: true,
            runs: 10,
          },
        },
      },

Recommendation:
It is recommended to increase optimizer runs.
*********************************************************************************
Tittle:
Variables need not be initialized to zero

Description:
The default value for variables is zero, so initializing them to zero is redundant.

    uint256 totalDepositAmount = 0;

https://github.com/code-423n4/2024-04-noya/blob/9c79b332eff82011dcfa1e8fd51bad805159d758/contracts/connectors/SiloConnector.sol#L109

*********************************************************************************
Tittle:
Typographical error in function name

Description:
The function name checkIfTVLHasDroped() in the AccountingManager.sol contract contains a typographical error, where "Droped" is misspelled as "Dropped". While being non-critical and not impacting the functionality, security, or performance of the contract, the typo could potentially cause mild confusion or readability issues for developers/users reviewing or interacting with the code. Correcting the spelling would enhance code clarity and maintain naming consistency without affecting the contract's execution or behavior.

*********************************************************************************
Tittle:
Use a specific version of Solidity instead of a wide version

Description:
Description: LockboxAdapterBlast.sol should use pragma solidity 0.8.13; instead of pragma solidity ^0.8.13;.
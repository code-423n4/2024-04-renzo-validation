## Front-runnable Initializers

* All contract initializers were missing access controls, allowing any user to initialize the contract. By front-running the contract deployers to initialize the contract, the incorrect parameters may be supplied, leaving the contract needing to be redeployed.

## Proof of Concept

Navigate to the following contracts:

 https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L75-L86

Initialize functions does not have access control. They are vulnerable to front-running.

## Recommended Mitigation Steps

* While the code that can be run in contract constructors is limited, setting the owner in the contract’s constructor to the msg.sender and adding the onlyOwner modifier to all initializers would be a sufficient level of access control.
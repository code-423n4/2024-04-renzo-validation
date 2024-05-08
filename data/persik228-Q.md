# Lines of code

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L77
https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L44


# Vulnerability details

## Impact
If Chainlink updates the routers to new addresses, [CCIPReceiver.sol](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol) and [xRenzoBridge.sol](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol) will stop working properly because these contracts lack logic to update the routers' addresses, which will also stop [xRenzoDeposit.sol's updatePrice()](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol#L310) function from working.

## Proof of Concept
As you can see on [Chainlink's docs](https://docs.chain.link/ccip/release-notes#v100-deprecated-on-mainnet---2024-04-01), if Chainlink's team updates the routers, the addresses of the routers will change and the previous ones will be deprecated.
> CCIP v1.0.0 is no longer supported on mainnet. You must use the new router addresses listed in the CCIP v1.2.0 configuration page.

For example, if a [CCIPReceiver](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol) can't update router's address, it will revert on new router's messages, which are price feeds from L1. That will cause L2's [xRenzoDeposit.sol](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol#L310) to use stale prices and put users and the protocol to risk.

## Tools Used

Manual review

## Recommended Mitigation Steps
Add a function to [CCIPReceiver.sol](https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol) that lets the owner update the router's address

## Title
getCollateralTokenIndex(_collateralToken) is not implemented in the `sweepERC20` function

## Impact
uint256 tokenIndex = getCollateralTokenIndex(_collateralToken); is not implemented in the `depositTokenRewardsFromProtocol`

an attacker can donate some erc20 tokens with either logic like fee on transfer or bad/corrupted logic with a similar token address to say stETH (proxy minting) in the hope that the rewards admin calls the sweepERC20 on that bad token without due diligence. This could lead to unexpected and complex issues

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L647

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L269
## Recommended Mitigation Steps
Add `uint256 tokenIndex = getCollateralTokenIndex(_collateralToken);` in the depositTokenRewardsFromProtocol in the restakeManager
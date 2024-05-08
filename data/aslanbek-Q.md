## L-01 `onlyNativeEthRestakeAdmin` modifier in `RewardHandler#forwardRewards` can be bypassed via 1 wei donation

In `RewardHandler`, both `receive` and `forwardRewards` implement the same logic:
```solidity
    /// @dev Forwards all native ETH rewards to the DepositQueue contract
    /// Handle ETH sent to this contract from outside the protocol that trigger contract execution - e.g. rewards
    receive() external payable nonReentrant {
        _forwardETH();
    }


    /// @dev Forwards all native ETH rewards to the DepositQueue contract
    /// Handle ETH sent to this contract from validator nodes that do not trigger contract execution - e.g. rewards
    function forwardRewards() external nonReentrant onlyNativeEthRestakeAdmin {
        _forwardETH();
    }
```
While `forwardRewards` is allowed to be called only by `NATIVE_ETH_RESTAKE_ADMIN`, `receive` can be triggered by anyone, which makes the modifier useless.

Consider adding the `onlyNativeEthRestakeAdmin` modifier to `receive`, or removing it from `forwardRewards`.
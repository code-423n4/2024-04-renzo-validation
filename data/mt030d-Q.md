
# 1. Incorrect Deposit Amount Logged in RestakeManager's Deposit Event

## Detail

```
emit Deposit(msg.sender, _collateralToken, _amount, ezETHToMint, _referralId);
```
The RestakeManager's [`deposit()`](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L491-L576) function emits a Deposit event, logging the amount of collateral tokens deposited by the user.

However, before emitting this event, the `_amount` variable is decreased if the depositQueue needs to be filled:
```
        if (bufferToFill > 0) {
            bufferToFill = (_amount <= bufferToFill) ? _amount : bufferToFill;
            // update amount to send to the operator Delegator
            _amount -= bufferToFill;
```
If `bufferToFill` is not zero, the `_amount` value passed into the Deposit event will be less than the actual deposited value from the user.

## Impact
If the deposit amount in the Deposit event is used to determine the user's ezPoints and further rewards (e.g., airdrop of $REZ tokens), this issue is significant. The Deposit event logs a lower deposited amount, which could lead to fewer platform rewards for the user.

# 2. `WithdrawQueue.claim()` may alter the order of `withdrawRequests`, causing the `withdrawRequestIndex` to mismatch the `withdrawRequestID` in `WithdrawRequestCreated` events.

```
        // delete the withdraw request
        withdrawRequests[msg.sender][withdrawRequestIndex] = withdrawRequests[msg.sender][
            withdrawRequests[msg.sender].length - 1
        ];
        withdrawRequests[msg.sender].pop();
```
The `WithdrawQueue.claim()` function swaps the last element in `withdrawRequests` with the indexed one and then removes the last element. As a result, the element at the `withdrawRequestID` of `WithdrawRequestCreated` events may not match the original element. Users should be aware of this potential issue.

# 3. Incorrect function name in `ConnextReceiver`
The function `updateCCIPEthChainSelector()` in the `ConnextReceiver` contract is incorrectly named. It interacts with `ConnextEthChainSelector`, not `CCIPEthChainSelector`. Therefore, we should rename this function to reflect its actual operation.

# 4. Inconsistency between role Name and responsibilities for nativeEthRestakeAdmin

From its name, one would assume that the nativeEthRestakeAdmin role is solely responsible for actions related to native ETH. However, this is not the case. The `OperatorDelegator.queueWithdrawals` and `OperatorDelegator.completeQueuedWithdrawal` functions, which are modified by onlyNativeEthRestakeAdmin, handle not just native ETH, but also LST withdrawals. This means that the nativeEthRestakeAdmin role also oversees the withdrawal of LST from EigenLayer using these functions.
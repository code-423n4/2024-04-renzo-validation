## Title
Incorrect Deposit Amount Logged in RestakeManager's Deposit Event

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
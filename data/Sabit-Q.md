1. depositTo() didn't check that "to" address is not the msg.sender

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L79

deposit() already allows the msg.sender to deposit and mint XERC20 to the msg.sender.

https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L69

So, the purpose of the depositTo() is to allow msg.sender deposit ERC20 tokens and then mint XERC20 token to a different "to" address. 

However, there's no check in depositTo not allowing the "to" address to be the msg.sender.

Recommendation:
Check that "to" address is not msg.sender:

require(to != msg.sender, "deposit to msg.sender");
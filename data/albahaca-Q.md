# Total Low Risk Issues : 30

# Total NC Issues : 93

# Total Gas Issues : 40

Note: i do not find that issues finded by bots

## L001 - Consider the case where `totalsupply` is 0:

Consider the case where `totalSupply` is 0. When `totalSupply` is 0, it should return 0 directly, because there will be an error of dividing by 0.


<details>
<summary>Click to show 4 findings</summary>

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


return (10 ** 18 * totalTVL) / totalSupply;

return (10 ** 18 * totalTVL) / totalSupply;

return (10 ** 18 * totalTVL) / totalSupply;

return (10 ** 18 * totalTVL) / totalSupply;

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L40:40

</details>

## L002 - `abi.encodePacked()` should not be used with dynamic types when passing the result to a hash function such as `keccak256()`:

Use `abi.encode()` instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. `abi.encodePacked(0x123,0x456)` => `0x123456` => `abi.encodePacked(0x1,0x23456)`, but `abi.encode(0x123,0x456)` => `0x0...1230...456`). Unless there is a compelling reason, `abi.encode` should be preferred. If there is only one argument to `abi.encodePacked()` it can often be cast to `bytes()` or `bytes32()` [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739). If all arguments are strings and or bytes, `bytes.concat()` should be used instead.


```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


bytes32 _salt = keccak256(abi.encodePacked(_name, _symbol, msg.sender));

bytes32 _salt = keccak256(abi.encodePacked(_xerc20, _baseToken, msg.sender));

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L190:190

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


bytes32 _salt = keccak256(abi.encodePacked(_name, _symbol, msg.sender));

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L89:89

## L003 - Prefer continue over revert model in iteration:

Preferably, it's better to skip operations on array indices when a condition is not met instead of reverting the entire transaction. Reverting could be exploited by malicious actors who might intentionally introduce array objects failing conditional checks, disrupting group operations. It's advisable to skip array indices rather than revert, unless there are valid security or logic reasons for doing otherwise


<details>
<summary>Click to show 4 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


218             for (uint256 i = 0; i < _destinationParam.length; ) {
219                 Client.EVM2AnyMessage memory evm2AnyMessage = Client.EVM2AnyMessage({
220                     receiver: abi.encode(_destinationParam[i]._renzoReceiver), // ABI-encoded xRenzoDepsot contract address
221                     data: _callData, // ABI-encoded ezETH exchange rate with Timestamp
222                     tokenAmounts: new Client.EVMTokenAmount[](0), // Empty array indicating no tokens are being sent
223                     extraArgs: Client._argsToBytes(
224                         // Additional arguments, setting gas limit
225                         Client.EVMExtraArgsV1({ gasLimit: 200_000 })
226                     ),
227                     // Set the feeToken  address, indicating LINK will be used for fees
228                     feeToken: address(linkToken)
229                 });
230     
231                 // Get the fee required to send the message
232                 uint256 fees = linkRouterClient.getFee(
233                     _destinationParam[i].destinationChainSelector,
234                     evm2AnyMessage
235                 );
236     
237                 if (fees > linkToken.balanceOf(address(this)))
238                     revert NotEnoughBalance(linkToken.balanceOf(address(this)), fees);
239     
240                 // approve the Router to transfer LINK tokens on contract's behalf. It will spend the fees in LINK
241                 linkToken.approve(address(linkRouterClient), fees);
242     
243                 // Send the message through the router and store the returned message ID
244                 bytes32 messageId = linkRouterClient.ccipSend(
245                     _destinationParam[i].destinationChainSelector,
246                     evm2AnyMessage
247                 );
248     
249                 // Emit an event with message details
250                 emit MessageSent(
251                     messageId,
252                     _destinationParam[i].destinationChainSelector,
253                     _destinationParam[i]._renzoReceiver,
254                     exchangeRate,
255                     address(linkToken),
256                     fees
257                 );
258                 unchecked {
259                     ++i;
260                 }
261             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


209             for (uint256 i; i < tokens.length; ) {
210                 if (address(tokens[i]) == IS_NATIVE) {
211                     // set beaconChainEthStrategy for ETH
212                     queuedWithdrawalParams[0].strategies[i] = eigenPodManager.beaconChainETHStrategy();
213     
214                     // set shares for ETH
215                     queuedWithdrawalParams[0].shares[i] = tokenAmounts[i];
216                 } else {
217                     if (address(tokenStrategyMapping[tokens[i]]) == address(0))
218                         revert InvalidZeroInput();
219     
220                     // set the strategy of the token
221                     queuedWithdrawalParams[0].strategies[i] = tokenStrategyMapping[tokens[i]];
222     
223                     // set the equivalent shares for tokenAmount
224                     queuedWithdrawalParams[0].shares[i] = tokenStrategyMapping[tokens[i]]
225                         .underlyingToSharesView(tokenAmounts[i]);
226                 }
227     
228                 // set withdrawer as this contract address
229                 queuedWithdrawalParams[0].withdrawer = address(this);
230     
231                 // track shares of tokens withdraw for TVL
232                 queuedShares[address(tokens[i])] += queuedWithdrawalParams[0].shares[i];
233                 unchecked {
234                     ++i;
235                 }
236             }
277             for (uint256 i; i < tokens.length; ) {
278                 if (address(tokens[i]) == address(0)) revert InvalidZeroInput();
279     
280                 // deduct queued shares for tracking TVL
281                 queuedShares[address(tokens[i])] -= withdrawal.shares[i];
282     
283                 // check if token is not Native ETH
284                 if (address(tokens[i]) != IS_NATIVE) {
285                     // Check the withdraw buffer and fill if below buffer target
286                     uint256 bufferToFill = withdrawQueue.getBufferDeficit(address(tokens[i]));
287     
288                     // get balance of this contract
289                     uint256 balanceOfToken = tokens[i].balanceOf(address(this));
290                     if (bufferToFill > 0) {
291                         bufferToFill = (balanceOfToken <= bufferToFill) ? balanceOfToken : bufferToFill;
292     
293                         // update amount to send to the operator Delegator
294                         balanceOfToken -= bufferToFill;
295     
296                         // safe Approve for depositQueue
297                         tokens[i].safeApprove(address(restakeManager.depositQueue()), bufferToFill);
298     
299                         // fill Withdraw Buffer via depositQueue
300                         restakeManager.depositQueue().fillERC20withdrawBuffer(
301                             address(tokens[i]),
302                             bufferToFill
303                         );
304                     }
305     
306                     // Deposit remaining tokens back to eigenLayer
307                     if (balanceOfToken > 0) {
308                         _deposit(tokens[i], balanceOfToken);
309                     }
310                 }
311                 unchecked {
312                     ++i;
313                 }
314             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


137             for (uint256 i = 0; i < odLength; ) {
138                 if (address(operatorDelegators[i]) == address(_newOperatorDelegator))
139                     revert AlreadyAdded();
140                 unchecked {
141                     ++i;
142                 }
143             }
223             for (uint256 i = 0; i < tokenLength; ) {
224                 if (address(collateralTokens[i]) == address(_newCollateralToken)) revert AlreadyAdded();
225                 unchecked {
226                     ++i;
227                 }
228             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


88              for (uint256 i = 0; i < _withdrawalBufferTarget.length; ) {
89                  if (
90                      _withdrawalBufferTarget[i].asset == address(0) ||
91                      _withdrawalBufferTarget[i].bufferAmount == 0
92                  ) revert InvalidZeroInput();
93                  withdrawalBufferTarget[_withdrawalBufferTarget[i].asset] = _withdrawalBufferTarget[i]
94                      .bufferAmount;
95                  unchecked {
96                      ++i;
97                  }
98              }
110             for (uint256 i = 0; i < _newBufferTarget.length; ) {
111                 if (_newBufferTarget[i].asset == address(0) || _newBufferTarget[i].bufferAmount == 0)
112                     revert InvalidZeroInput();
113                 emit WithdrawBufferTargetUpdated(
114                     withdrawalBufferTarget[_newBufferTarget[i].asset],
115                     _newBufferTarget[i].bufferAmount
116                 );
117                 withdrawalBufferTarget[_newBufferTarget[i].asset] = _newBufferTarget[i].bufferAmount;
118                 unchecked {
119                     ++i;
120                 }
121             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## L004 - Consider disallowing minting/transfers to address(this):

A tranfer to the token contract itself is unlikely to be correct and more likely to be a common user error due to a copy & paste mistake. Proceeding with such a transfer will result in the permanent loss of user tokens.


```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


96          function mint(address _user, uint256 _amount) public virtual {
97              _mintWithCaller(msg.sender, _user, _amount);
98          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


64          function mint(address _to, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
65              XERC20.mint(_to, _amount);
66          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


41          function mint(address to, uint256 amount) external onlyMinterBurner {
42              _mint(to, amount);
43          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

## L005 - External calls in modifiers should be avoided:

Using external calls within modifiers can introduce reentrancy risks and make a contract's logic less transparent. Modifiers are meant for pre-execution checks, and external calls can lead to unpredictable flow and potential reentrancy issues. Avoiding external calls in modifiers and placing them directly in the function body enhances code clarity, aids in auditing, and minimizes unexpected behaviors. This practice ensures a more explicit execution order and clearer understanding of the code's effects.


```solidity
File: contracts/Withdraw/WithdrawQueue.sol


51              if (msg.sender != address(restakeManager.depositQueue())) revert NotDepositQueue();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0



## L006 - Loss of precision due to division by large numbers:

Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator.


<details>
<summary>Click to show 4 findings</summary>

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


253             uint256 xezETHAmount = (1e18 * amountOut) / _lastPrice;
282                 return (sweepBatchSize * bridgeFeeShare) / FEE_BASIS;
280                 return (_amountIn * bridgeFeeShare) / FEE_BASIS;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


158             uint256 redeemAmount = (_currentValueInProtocol * _ezETHBeingBurned) / _existingEzETHSupply;
80              return (uint256(price) * _balance) / SCALE_FACTOR;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


40              return (10 ** 18 * totalTVL) / totalSupply;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


379                     (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
380                         BASIS_POINTS /
379                     (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
380                         BASIS_POINTS /
381                         BASIS_POINTS
421                     (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
422                         BASIS_POINTS /
423                         BASIS_POINTS &&
421                     (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
422                         BASIS_POINTS /


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

</details>

## L007 - Array lengths not checked:

If the length of the arrays are not required to be of the same length, user operations may not be fully executed due to a mismatch in the number of items iterated over, versus the number of items provided in the second array.


<details>
<summary>Click to show 5 findings</summary>

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


74          function deployXERC20(
75              string memory _name,
76              string memory _symbol,
77              uint256[] memory _minterLimits,
78              uint256[] memory _burnerLimits,
79              address[] memory _bridges,
80              address _proxyAdmin
81          ) external returns (address _xerc20) {
82              _xerc20 = _deployXERC20(
83                  _name,
84                  _symbol,
85                  _minterLimits,
86                  _burnerLimits,
87                  _bridges,
88                  _proxyAdmin
89              );
90      
91              emit XERC20Deployed(_xerc20);
92          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


37          function deployOptimismMintableXERC20(
38              string memory _name,
39              string memory _symbol,
40              uint256[] memory _minterLimits,
41              uint256[] memory _burnerLimits,
42              address[] memory _bridges,
43              address _proxyAdmin,
44              address _l1Token
45          ) public returns (address _xerc20) {
46              _xerc20 = _deployOptimismMintableXERC20(
47                  _name,
48                  _symbol,
49                  _minterLimits,
50                  _burnerLimits,
51                  _bridges,
52                  _proxyAdmin,
53                  _l1Token
54              );
55      
56              emit XERC20Deployed(_xerc20);
57          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


364         function verifyWithdrawalCredentials(
365             uint64 oracleTimestamp,
366             BeaconChainProofs.StateRootProof calldata stateRootProof,
367             uint40[] calldata validatorIndices,
368             bytes[] calldata withdrawalCredentialProofs,
369             bytes32[][] calldata validatorFields
370         ) external onlyNativeEthRestakeAdmin {
371             uint256 gasBefore = gasleft();
372             eigenPod.verifyWithdrawalCredentials(
373                 oracleTimestamp,
374                 stateRootProof,
375                 validatorIndices,
376                 withdrawalCredentialProofs,
377                 validatorFields
378             );
379     
380             // Decrement the staked but not verified ETH
381             for (uint256 i = 0; i < validatorFields.length; ) {
382                 uint64 validatorCurrentBalanceGwei = BeaconChainProofs.getEffectiveBalanceGwei(
383                     validatorFields[i]
384                 );
385                 stakedButNotVerifiedEth -= (validatorCurrentBalanceGwei * GWEI_TO_WEI);
386                 unchecked {
387                     ++i;
388                 }
389             }
390             // update the gas spent for RestakeAdmin
391             _recordGas(gasBefore);
392         }
405         function verifyAndProcessWithdrawals(
406             uint64 oracleTimestamp,
407             BeaconChainProofs.StateRootProof calldata stateRootProof,
408             BeaconChainProofs.WithdrawalProof[] calldata withdrawalProofs,
409             bytes[] calldata validatorFieldsProofs,
410             bytes32[][] calldata validatorFields,
411             bytes32[][] calldata withdrawalFields
412         ) external onlyNativeEthRestakeAdmin {
413             uint256 gasBefore = gasleft();
414             eigenPod.verifyAndProcessWithdrawals(
415                 oracleTimestamp,
416                 stateRootProof,
417                 withdrawalProofs,
418                 validatorFieldsProofs,
419                 validatorFields,
420                 withdrawalFields
421             );
422             // update the gas spent for RestakeAdmin
423             _recordGas(gasBefore);
424         }
446         function recoverTokens(
447             IERC20[] memory tokenList,
448             uint256[] memory amountsToWithdraw,
449             address recipient
450         ) external onlyNativeEthRestakeAdmin {
451             eigenPod.recoverTokens(tokenList, amountsToWithdraw, recipient);
452         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


400         function chooseOperatorDelegatorForWithdraw(
401             uint256 tokenIndex,
402             uint256 ezETHValue,
403             uint256[][] memory operatorDelegatorTokenTVLs,
404             uint256[] memory operatorDelegatorTVLs,
405             uint256 totalTVL
406         ) public view returns (IOperatorDelegator) {
407             // If there is only one operator delegator, try to use it
408             if (operatorDelegators.length == 1) {
409                 // If the OD doesn't have the tokens, revert
410                 if (operatorDelegatorTokenTVLs[0][tokenIndex] < ezETHValue) {
411                     revert NotFound();
412                 }
413                 return operatorDelegators[0];
414             }
415     
416             // Fnd the operator delegator with TVL above the threshold and with enough tokens
417             uint256 odLength = operatorDelegatorTVLs.length;
418             for (uint256 i = 0; i < odLength; ) {
419                 if (
420                     operatorDelegatorTVLs[i] >
421                     (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
422                         BASIS_POINTS /
423                         BASIS_POINTS &&
424                     operatorDelegatorTokenTVLs[i][tokenIndex] >= ezETHValue
425                 ) {
426                     return operatorDelegators[i];
427                 }
428     
429                 unchecked {
430                     ++i;
431                 }
432             }
433     
434             // If not found, just find one with enough tokens
435             for (uint256 i = 0; i < odLength; ) {
436                 if (operatorDelegatorTokenTVLs[i][tokenIndex] >= ezETHValue) {
437                     return operatorDelegators[i];
438                 }
439     
440                 unchecked {
441                     ++i;
442                 }
443             }
444     
445             // This token cannot be withdrawn
446             revert NotFound();
447         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/TimelockController.sol


215         function hashOperationBatch(
216             address[] calldata targets,
217             uint256[] calldata values,
218             bytes[] calldata payloads,
219             bytes32 predecessor,
220             bytes32 salt
221         ) public pure virtual returns (bytes32) {
222             return keccak256(abi.encode(targets, values, payloads, predecessor, salt));
223         }
436         function onERC1155BatchReceived(
437             address,
438             address,
439             uint256[] memory,
440             uint256[] memory,
441             bytes memory
442         ) public virtual override returns (bytes4) {
443             return this.onERC1155BatchReceived.selector;
444         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

</details>

## L008 - Missing checks for address(0x0) when updating address state variables

issing checks for address(0x0) when updating address state variables


<details>
<summary>Click to show 6 findings</summary>

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


138             receiver = _receiver;
513             receiver = _receiver;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


86              FACTORY = _factory;
123             lockbox = _lockbox;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


58              lockboxImplementation = _lockboxImplementation;
59              xerc20Implementation = _xerc20Implementation;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


44              l1Token = _l1Token;
45              optimismBridge = _optimismBridge;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


125             delegateAddress = _delegateAddress;
229                 queuedWithdrawalParams[0].withdrawer = address(this);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


45              rewardDestination = _rewardDestination;
77              rewardDestination = _rewardDestination;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

</details>

## L009 - Missing checks for `address(0x0)` when assigning values to address state variables:

This issue arises when an address state variable is assigned a value without a preceding check to ensure it isn't address(0x0). This can lead to unexpected behavior as address(0x0) often represents an uninitialized address.


<details>
<summary>Click to show 4 findings</summary>

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


86              FACTORY = _factory;
123             lockbox = _lockbox;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


58              lockboxImplementation = _lockboxImplementation;
59              xerc20Implementation = _xerc20Implementation;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


45              XERC20 = IXERC20(_xerc20);
46              ERC20 = IERC20(_erc20);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


44              l1Token = _l1Token;
45              optimismBridge = _optimismBridge;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

</details>

## L010 - Missing checks for the L2 Sequencer grace period:

Chainlink recommends that users using price oracles, check whether the grace period has passed in the event the Arbitrum Sequencer [goes down](https://docs.chain.link/data-feeds/l2-sequencer-feeds#arbitrum). To help your applications identify when the sequencer is unavailable, you can use a data feed that tracks the last known status of the sequencer at a given point in time. This helps you prevent mass liquidations by providing a grace period to allow customers to react to such an event.


```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


50          function getMintRate() public view returns (uint256, uint256) {
51              (, int256 price, , uint256 timestamp, ) = oracle.latestRoundData();
52              if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();
53              // scale the price to have 18 decimals
54              uint256 _scaledPrice = (uint256(price)) * 10 ** (18 - oracle.decimals());
55              if (_scaledPrice < 1 ether) revert InvalidOraclePrice();
56              return (_scaledPrice, timestamp);
57          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


71          function lookupTokenValue(IERC20 _token, uint256 _balance) public view returns (uint256) {
72              AggregatorV3Interface oracle = tokenOracleLookup[_token];
73              if (address(oracle) == address(0x0)) revert OracleNotFound();
74      
75              (, int256 price, , uint256 timestamp, ) = oracle.latestRoundData();
76              if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();
77              if (price <= 0) revert InvalidOraclePrice();
78      
79              // Price is times 10**18 ensure value amount is scaled
80              return (uint256(price) * _balance) / SCALE_FACTOR;
81          }
85          function lookupTokenAmountFromValue(
86              IERC20 _token,
87              uint256 _value
88          ) external view returns (uint256) {
89              AggregatorV3Interface oracle = tokenOracleLookup[_token];
90              if (address(oracle) == address(0x0)) revert OracleNotFound();
91      
92              (, int256 price, , uint256 timestamp, ) = oracle.latestRoundData();
93              if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();
94              if (price <= 0) revert InvalidOraclePrice();
95      
96              // Price is times 10**18 ensure token amount is scaled
97              return (_value * SCALE_FACTOR) / uint256(price);
98          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0




## L011 - Return values of `approve()` not checked:

Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything


```solidity
File: contracts/Deposits/DepositQueue.sol


268                 token.approve(address(restakeManager), balance - feeAmount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

## L012 - Large transfers may not work with some ERC20 tokens:

Some  IERC20 implementations (e.g UNI, COMP) may fail if the valued transferred is larger than uint96. [Source](https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers)


<details>
<summary>Click to show 8 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


83              SafeERC20.safeTransferFrom(IERC20(_erc20), msg.sender, address(this), _amount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


306             IERC20(_token).safeTransfer(_to, _amount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


214             depositToken.safeTransferFrom(msg.sender, address(this), _amountIn);
490             IERC20(_token).safeTransfer(_to, _amount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


147                 ERC20.safeTransferFrom(msg.sender, address(this), _amount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


151             token.safeTransferFrom(msg.sender, address(this), tokenAmount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


140             IERC20(_asset).safeTransferFrom(msg.sender, address(this), _amount);
262                     IERC20(token).safeTransfer(feeAddress, feeAmount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


540             _collateralToken.safeTransferFrom(msg.sender, address(this), _amount);
661             _token.safeTransferFrom(msg.sender, address(this), _amount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


197             IERC20(_asset).safeTransferFrom(msg.sender, address(this), _amount);
214             IERC20(address(ezETH)).safeTransferFrom(msg.sender, address(this), _amount);
305                 IERC20(_withdrawRequest.collateralToken).transfer(
306                     msg.sender,
307                     _withdrawRequest.amountToRedeem
308                 );


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## L013 - Large approvals may not work with some tokens:

Not all IERC20 implementations are totally compliant, and some (e.g UNI, COMP) may fail if the valued passed is larger than uint96. [Source](https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers)


```solidity
File: contracts/Deposits/DepositQueue.sol


268                 token.approve(address(restakeManager), balance - feeAmount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

## L014 - Setters should have initial value check:

Setters should have initial value check to prevent assigning wrong value to the variable. Assginment of wrong value can lead to unexpected behavior of the contract.


<details>
<summary>Click to show 9 findings</summary>

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


36          function setOracleAddress(AggregatorV3Interface _oracleAddress) external onlyOwner {
37              if (address(_oracleAddress) == address(0)) revert InvalidZeroInput();
38              // Verify that the pricing of the oracle is less than or equal to 18 decimals - pricing calculations will be off otherwise
39              if (_oracleAddress.decimals() > 18)
40                  revert InvalidTokenDecimals(18, _oracleAddress.decimals());
41      
42              emit OracleAddressUpdated(address(_oracleAddress), address(oracle));
43              oracle = _oracleAddress;
44          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


466         function setAllowedBridgeSweeper(address _sweeper, bool _allowed) external onlyOwner {
467             allowedBridgeSweepers[_sweeper] = _allowed;
468     
469             emit BridgeSweeperAddressUpdated(_sweeper, _allowed);
470         }
501         function setOraclePriceFeed(IRenzoOracleL2 _oracle) external onlyOwner {
502             emit OraclePriceFeedUpdated(address(_oracle), address(oracle));
503             oracle = _oracle;
504         }
511         function setReceiverPriceFeed(address _receiver) external onlyOwner {
512             emit ReceiverPriceFeedUpdated(_receiver, receiver);
513             receiver = _receiver;
514         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


121         function setLockbox(address _lockbox) public {
122             if (msg.sender != FACTORY) revert IXERC20_NotFactory();
123             lockbox = _lockbox;
124     
125             emit LockboxSet(_lockbox);
126         }
135         function setLimits(
136             address _bridge,
137             uint256 _mintingLimit,
138             uint256 _burningLimit
139         ) external onlyOwner {
140             _changeMinterLimit(_bridge, _mintingLimit);
141             _changeBurnerLimit(_bridge, _burningLimit);
142             emit BridgeLimitsSet(_mintingLimit, _burningLimit, _bridge);
143         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


106         function setTokenStrategy(
107             IERC20 _token,
108             IStrategy _strategy
109         ) external nonReentrant onlyOperatorDelegatorAdmin {
110             if (address(_token) == address(0x0)) revert InvalidZeroInput();
111     
112             tokenStrategyMapping[_token] = _strategy;
113             emit TokenStrategyUpdated(_token, _strategy);
114         }
117         function setDelegateAddress(
118             address _delegateAddress,
119             ISignatureUtils.SignatureWithExpiry memory approverSignatureAndExpiry,
120             bytes32 approverSalt
121         ) external nonReentrant onlyOperatorDelegatorAdmin {
122             if (address(_delegateAddress) == address(0x0)) revert InvalidZeroInput();
123             if (address(delegateAddress) != address(0x0)) revert DelegateAddressAlreadySet();
124     
125             delegateAddress = _delegateAddress;
126     
127             delegationManager.delegateTo(delegateAddress, approverSignatureAndExpiry, approverSalt);
128     
129             emit DelegationAddressUpdated(_delegateAddress);
130         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


87          function setWithdrawQueue(IWithdrawQueue _withdrawQueue) external onlyRestakeManagerAdmin {
88              if (address(_withdrawQueue) == address(0)) revert InvalidZeroInput();
89              emit WithdrawQueueUpdated(address(withdrawQueue), address(_withdrawQueue));
90              withdrawQueue = _withdrawQueue;
91          }
112         function setRestakeManager(IRestakeManager _restakeManager) external onlyRestakeManagerAdmin {
113             if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
114     
115             restakeManager = _restakeManager;
116     
117             emit RestakeManagerUpdated(_restakeManager);
118         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


54          function setOracleAddress(
55              IERC20 _token,
56              AggregatorV3Interface _oracleAddress
57          ) external nonReentrant onlyOracleAdmin {
58              if (address(_token) == address(0x0)) revert InvalidZeroInput();
59      
60              // Verify that the pricing of the oracle is 18 decimals - pricing calculations will be off otherwise
61              if (_oracleAddress.decimals() != 18)
62                  revert InvalidTokenDecimals(18, _oracleAddress.decimals());
63      
64              tokenOracleLookup[_token] = _oracleAddress;
65              emit OracleAddressUpdated(_token, _oracleAddress);
66          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


121         function setPaused(bool _paused) external onlyDepositWithdrawPauserAdmin {
122             paused = _paused;
123         }
215         function setMaxDepositTVL(uint256 _maxDepositTVL) external onlyRestakeManagerAdmin {
216             maxDepositTVL = _maxDepositTVL;
217         }
709         function setTokenTvlLimit(IERC20 _token, uint256 _limit) external onlyRestakeManagerAdmin {
710             // Verify collateral token is in the list - call will revert if not found
711             getCollateralTokenIndex(_token);
712     
713             // Set the limit
714             collateralTokenTvlLimits[_token] = _limit;
715     
716             emit CollateralTokenTvlUpdated(_token, _limit);
717         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


72          function setRewardDestination(
73              address _rewardDestination
74          ) external nonReentrant onlyRestakeManagerAdmin {
75              if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();
76      
77              rewardDestination = _rewardDestination;
78      
79              emit RewardDestinationUpdated(_rewardDestination);
80          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


51          function setPaused(bool _paused) external onlyTokenAdmin {
52              paused = _paused;
53          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## L015 - Code does not follow the best practice of check-effects-interaction:

Code should follow the best-practice of [check-effects-interaction](https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-11-coding-patterns/topic/checks-effects-interactions/), where state variables are updated before any external calls are made. Doing so prevents a large class of reentrancy bugs.


```solidity
File: contracts/Deposits/DepositQueue.sol


/// @audit restakeManager.depositTokenRewardsFromProtocol() called prior to this assignment
272                 totalEarned[address(token)] = totalEarned[address(token)] + balance - feeAmount;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L272:272

```solidity
File: contracts/TimelockController.sol


/// @audit _execute() called prior to this assignment
/// @audit contains nested function call target.call() 
/// @audit located in file contest/contracts/TimelockController.sol  
389             _timestamps[id] = _DONE_TIMESTAMP;


/// @audit _execute() called prior to this assignment
/// @audit contains nested function call target.call() 
/// @audit located in file contest/contracts/TimelockController.sol  
389             _timestamps[id] = _DONE_TIMESTAMP;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L389:389

## L016 - Contracts are designed to receive ETH but do not implement function for withdrawal:

The following contracts can receive ETH but can not withdraw, ETH is occasionally sent by users will be stuck in those contracts. This functionality also applies to baseTokens resulting in locked tokens and loss of funds.


```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


313         receive() external payable {}


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


542         receive() external payable {}


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/TimelockController.sol


137         receive() external payable {}


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

## L017 - Functions calling contracts/addresses with transfer hooks are missing reentrancy guards:

Even if the function follows the best practice of check-effects-interaction, not using a reentrancy guard when there may be transfer hooks to unknown or untrusted ERC20 tokens will open the users of this protocol up to [read-only reentrancies](https://chainsecurity.com/curve-lp-oracle-manipulation-post-mortem/) with no way to protect against it, except by block-listing the whole protocol.


<details>
<summary>Click to show 4 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


83              SafeERC20.safeTransferFrom(IERC20(_erc20), msg.sender, address(this), _amount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


306             IERC20(_token).safeTransfer(_to, _amount);
295             payable(_to).transfer(_amount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


479             payable(_to).transfer(_amount);
490             IERC20(_token).safeTransfer(_to, _amount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


262                     IERC20(token).safeTransfer(feeAddress, feeAmount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

</details>

## L018 - Burn functions should be protected with a modifier:

  


```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


107         function burn(address _user, uint256 _amount) public virtual {
108             if (msg.sender != _user) {
109                 _spendAllowance(_user, msg.sender, _amount);
110             }
111     
112             _burnWithCaller(msg.sender, _user, _amount);
113         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


68          function burn(address _from, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
69              XERC20.burn(_from, _amount);
70          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

## L019 - Constant decimal values:

The use of fixed decimal values such as 1e18 or 1e8 in Solidity contracts can lead to inaccuracies, bugs, and vulnerabilities, particularly when interacting with tokens having different decimal configurations. Not all ERC20 tokens follow the standard 18 decimal places, and assumptions about decimal places can lead to miscalculations. Always retrieve and use the decimals() function from the token contract itself when performing calculations involving token amounts.


```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


253             uint256 xezETHAmount = (1e18 * amountOut) / _lastPrice;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

## L020 - No limits when setting state variable amounts:

It is important to ensure state variables numbers are set to a reasonable value.


```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


233             bridges[_bridge].minterParams.maxLimit = _limit;
255             bridges[_bridge].burnerParams.maxLimit = _limit;
308             _limit = _currentLimit;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


216             maxDepositTVL = _maxDepositTVL;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/TimelockController.sol


117             _minDelay = minDelay;
405             _minDelay = newDelay;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

## L021 - Allowed fees/rates should be capped by smart contracts:

Fees/rates should be required to be below 100%, preferably at a much lower limit, to ensure users don't have to monitor the blockchain for changes prior to using the protocol.


```solidity
File: contracts/Delegation/OperatorDelegator.sol


106         function setTokenStrategy(
107             IERC20 _token,
108             IStrategy _strategy
109         ) external nonReentrant onlyOperatorDelegatorAdmin {
110             if (address(_token) == address(0x0)) revert InvalidZeroInput();
111     
112             tokenStrategyMapping[_token] = _strategy;
113             emit TokenStrategyUpdated(_token, _strategy);
114         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


93          function setFeeConfig(
94              address _feeAddress,
95              uint256 _feeBasisPoints
96          ) external onlyRestakeManagerAdmin {
97              // Verify address is set if basis points are non-zero
98              if (_feeBasisPoints > 0) {
99                  if (_feeAddress == address(0x0)) revert InvalidZeroInput();
100             }
101     
102             // Verify basis points are not over 100%
103             if (_feeBasisPoints > 10000) revert OverMaxBasisPoints();
104     
105             feeAddress = _feeAddress;
106             feeBasisPoints = _feeBasisPoints;
107     
108             emit FeeConfigUpdated(_feeAddress, _feeBasisPoints);
109         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

## L022 - Governance functions should be controlled by time locks:

Governance functions (such as upgrading contracts, setting critical parameters) should be controlled using time locks to introduce a delay between a proposal and its execution. This gives users time to exit before a potentially dangerous or malicious operation is applied.


<details>
<summary>Click to show 23 findings</summary>

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


36          function setOracleAddress(AggregatorV3Interface _oracleAddress) external onlyOwner {

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L36:44

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


107         function updateCCIPEthChainSelector(uint64 _newChainSelector) external onlyOwner {

125         function pause() external onlyOwner {

134         function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {

117         function unPause() external onlyOwner {

96          function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L96:100

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


121         function pause() external onlyOwner {

92          function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {

103         function updateCCIPEthChainSelector(uint32 _newChainDomain) external onlyOwner {

130         function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {

113         function unPause() external onlyOwner {

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L113:115

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


489         function recoverERC20(address _token, uint256 _amount, address _to) external onlyOwner {

521         function updateBridgeFeeShare(uint256 _newShare) external onlyOwner {

478         function recoverNative(uint256 _amount, address _to) external onlyOwner {

501         function setOraclePriceFeed(IRenzoOracleL2 _oracle) external onlyOwner {

532         function updateSweepBatchSize(uint256 _newBatchSize) external onlyOwner {

511         function setReceiverPriceFeed(address _receiver) external onlyOwner {

466         function setAllowedBridgeSweeper(address _sweeper, bool _allowed) external onlyOwner {

320         function updatePriceByOwner(uint256 price) external onlyOwner {

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L320:322

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


135         function setLimits(
136             address _bridge,
137             uint256 _mintingLimit,
138             uint256 _burningLimit
139         ) external onlyOwner {

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L135:143

```solidity
File: contracts/TimelockController.sol


259         function scheduleBatch(
260             address[] calldata targets,
261             uint256[] calldata values,
262             bytes[] calldata payloads,
263             bytes32 predecessor,
264             bytes32 salt,
265             uint256 delay
266         ) public virtual onlyRole(PROPOSER_ROLE) {

234         function schedule(
235             address target,
236             uint256 value,
237             bytes calldata data,
238             bytes32 predecessor,
239             bytes32 salt,
240             uint256 delay
241         ) public virtual onlyRole(PROPOSER_ROLE) {

296         function cancel(bytes32 id) public virtual onlyRole(CANCELLER_ROLE) {

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L296:301

</details>

## L023 - constructor/initialize function lacks parameter validation:

In Solidity, when values are being assigned in constructors to unsigned or integer variables, it's crucial to ensure the provided values adhere to the protocol's specific operational boundaries as laid out in the project specifications and documentation. If the constructors lack appropriate validation checks, there's a risk of setting state variables with values that could cause unexpected and potentially detrimental behavior within the contract's operations, violating the intended logic of the protocol. This can compromise the contract's security and impact the maintainability and reliability of the system. In order to avoid such issues, it is recommended to incorporate rigorous validation checks in constructors. These checks should align with the project's defined rules and constraints, making use of Solidity's built-in require function to enforce these conditions. If the validation checks fail, the require function will cause the transaction to revert, ensuring the integrity and adherence to the protocol's expected behavior.


<details>
<summary>Click to show 5 findings</summary>

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


61          function initialize(
62              string memory _name,
63              string memory _symbol,
64              address _factory
65          ) public initializer {
66              __XERC20_init(_name, _symbol, _factory);
67          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


54          function initialize(
55              address _lockboxImplementation,
56              address _xerc20Implementation
57          ) public initializer {
58              lockboxImplementation = _lockboxImplementation;
59              xerc20Implementation = _xerc20Implementation;
60          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


44          function initialize(address _xerc20, address _erc20, bool _isNative) public initializer {
45              XERC20 = IXERC20(_xerc20);
46              ERC20 = IERC20(_erc20);
47              IS_NATIVE = _isNative;
48          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


35          function initialize(
36              string memory _name,
37              string memory _symbol,
38              address _factory,
39              address _l1Token,
40              address _optimismBridge
41          ) public initializer {
42              __ERC165_init();
43              __XERC20_init(_name, _symbol, _factory);
44              l1Token = _l1Token;
45              optimismBridge = _optimismBridge;
46          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


101         function initialize(
102             IRoleManager _roleManager,
103             IEzEthToken _ezETH,
104             IRenzoOracle _renzoOracle,
105             IStrategyManager _strategyManager,
106             IDelegationManager _delegationManager,
107             IDepositQueue _depositQueue
108         ) public initializer {
109             __ReentrancyGuard_init();
110     
111             roleManager = _roleManager;
112             ezETH = _ezETH;
113             renzoOracle = _renzoOracle;
114             strategyManager = _strategyManager;
115             delegationManager = _delegationManager;
116             depositQueue = _depositQueue;
117             paused = false;
118         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

</details>

## L024 - Consider the case where `totalsupply` is 0:

Consider the case where `totalSupply` is 0. When `totalSupply` is 0, it should return 0 directly, because there will be an error of dividing by 0.


<details>
<summary>Click to show 4 findings</summary>

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


return (10 ** 18 * totalTVL) / totalSupply;

return (10 ** 18 * totalTVL) / totalSupply;

return (10 ** 18 * totalTVL) / totalSupply;

return (10 ** 18 * totalTVL) / totalSupply;

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L40:40

</details>

## L025 - For loops in `public` or `external` functions should be avoided due to high gas costs and possible DOS:

In Solidity, for loops can potentially cause Denial of Service (DoS) attacks if not handled carefully. DoS attacks can occur when an attacker intentionally exploits the gas cost of a function, causing it to run out of gas or making it too expensive for other users to call. Below are some scenarios where for loops can lead to DoS attacks: Nested for loops can become exceptionally gas expensive and should be used sparingly.


<details>
<summary>Click to show 6 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


210         function sendPrice(
211             CCIPDestinationParam[] calldata _destinationParam,
212             ConnextDestinationParam[] calldata _connextDestinationParam
213         ) external payable onlyPriceFeedSender nonReentrant {
214             // call getRate() to get the current price of ezETH
215             uint256 exchangeRate = rateProvider.getRate();
216             bytes memory _callData = abi.encode(exchangeRate, block.timestamp);
217             // send price feed to renzo CCIP receivers
218             for (uint256 i = 0; i < _destinationParam.length; ) {
219                 Client.EVM2AnyMessage memory evm2AnyMessage = Client.EVM2AnyMessage({
220                     receiver: abi.encode(_destinationParam[i]._renzoReceiver), // ABI-encoded xRenzoDepsot contract address
221                     data: _callData, // ABI-encoded ezETH exchange rate with Timestamp
222                     tokenAmounts: new Client.EVMTokenAmount[](0), // Empty array indicating no tokens are being sent
223                     extraArgs: Client._argsToBytes(
224                         // Additional arguments, setting gas limit
225                         Client.EVMExtraArgsV1({ gasLimit: 200_000 })
226                     ),
227                     // Set the feeToken  address, indicating LINK will be used for fees
228                     feeToken: address(linkToken)
229                 });
230     
231                 // Get the fee required to send the message
232                 uint256 fees = linkRouterClient.getFee(
233                     _destinationParam[i].destinationChainSelector,
234                     evm2AnyMessage
235                 );
236     
237                 if (fees > linkToken.balanceOf(address(this)))
238                     revert NotEnoughBalance(linkToken.balanceOf(address(this)), fees);
239     
240                 // approve the Router to transfer LINK tokens on contract's behalf. It will spend the fees in LINK
241                 linkToken.approve(address(linkRouterClient), fees);
242     
243                 // Send the message through the router and store the returned message ID
244                 bytes32 messageId = linkRouterClient.ccipSend(
245                     _destinationParam[i].destinationChainSelector,
246                     evm2AnyMessage
247                 );
248     
249                 // Emit an event with message details
250                 emit MessageSent(
251                     messageId,
252                     _destinationParam[i].destinationChainSelector,
253                     _destinationParam[i]._renzoReceiver,
254                     exchangeRate,
255                     address(linkToken),
256                     fees
257                 );
258                 unchecked {
259                     ++i;
260                 }
261             }
262     
263             // send price feed to renzo connext receiver
264             for (uint256 i = 0; i < _connextDestinationParam.length; ) {
265                 connext.xcall{ value: _connextDestinationParam[i].relayerFee }(
266                     _connextDestinationParam[i].destinationDomainId,
267                     _connextDestinationParam[i]._renzoReceiver,
268                     address(0),
269                     msg.sender,
270                     0,
271                     0,
272                     _callData
273                 );
274     
275                 emit ConnextMessageSent(
276                     _connextDestinationParam[i].destinationDomainId,
277                     _connextDestinationParam[i]._renzoReceiver,
278                     exchangeRate,
279                     _connextDestinationParam[i].relayerFee
280                 );
281     
282                 unchecked {
283                     ++i;
284                 }
285             }
286         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


193         function queueWithdrawals(
194             IERC20[] calldata tokens,
195             uint256[] calldata tokenAmounts
196         ) external nonReentrant onlyNativeEthRestakeAdmin returns (bytes32) {
197             // record gas spent
198             uint256 gasBefore = gasleft();
199             if (tokens.length != tokenAmounts.length) revert MismatchedArrayLengths();
200             IDelegationManager.QueuedWithdrawalParams[]
201                 memory queuedWithdrawalParams = new IDelegationManager.QueuedWithdrawalParams[](1);
202             // set strategies legth for 0th index only
203             queuedWithdrawalParams[0].strategies = new IStrategy[](tokens.length);
204             queuedWithdrawalParams[0].shares = new uint256[](tokens.length);
205     
206             // Save the nonce before starting the withdrawal
207             uint96 nonce = uint96(delegationManager.cumulativeWithdrawalsQueued(address(this)));
208     
209             for (uint256 i; i < tokens.length; ) {
210                 if (address(tokens[i]) == IS_NATIVE) {
211                     // set beaconChainEthStrategy for ETH
212                     queuedWithdrawalParams[0].strategies[i] = eigenPodManager.beaconChainETHStrategy();
213     
214                     // set shares for ETH
215                     queuedWithdrawalParams[0].shares[i] = tokenAmounts[i];
216                 } else {
217                     if (address(tokenStrategyMapping[tokens[i]]) == address(0))
218                         revert InvalidZeroInput();
219     
220                     // set the strategy of the token
221                     queuedWithdrawalParams[0].strategies[i] = tokenStrategyMapping[tokens[i]];
222     
223                     // set the equivalent shares for tokenAmount
224                     queuedWithdrawalParams[0].shares[i] = tokenStrategyMapping[tokens[i]]
225                         .underlyingToSharesView(tokenAmounts[i]);
226                 }
227     
228                 // set withdrawer as this contract address
229                 queuedWithdrawalParams[0].withdrawer = address(this);
230     
231                 // track shares of tokens withdraw for TVL
232                 queuedShares[address(tokens[i])] += queuedWithdrawalParams[0].shares[i];
233                 unchecked {
234                     ++i;
235                 }
236             }
237     
238             // queue withdrawal in EigenLayer
239             bytes32 withdrawalRoot = delegationManager.queueWithdrawals(queuedWithdrawalParams)[0];
240             // Emit the withdrawal started event
241             emit WithdrawStarted(
242                 withdrawalRoot,
243                 address(this),
244                 delegateAddress,
245                 address(this),
246                 nonce,
247                 block.number,
248                 queuedWithdrawalParams[0].strategies,
249                 queuedWithdrawalParams[0].shares
250             );
251     
252             // update the gas spent for RestakeAdmin
253             _recordGas(gasBefore);
254     
255             return withdrawalRoot;
256         }
265         function completeQueuedWithdrawal(
266             IDelegationManager.Withdrawal calldata withdrawal,
267             IERC20[] calldata tokens,
268             uint256 middlewareTimesIndex
269         ) external nonReentrant onlyNativeEthRestakeAdmin {
270             uint256 gasBefore = gasleft();
271             if (tokens.length != withdrawal.strategies.length) revert MismatchedArrayLengths();
272     
273             // complete the queued withdrawal from EigenLayer with receiveAsToken set to true
274             delegationManager.completeQueuedWithdrawal(withdrawal, tokens, middlewareTimesIndex, true);
275     
276             IWithdrawQueue withdrawQueue = restakeManager.depositQueue().withdrawQueue();
277             for (uint256 i; i < tokens.length; ) {
278                 if (address(tokens[i]) == address(0)) revert InvalidZeroInput();
279     
280                 // deduct queued shares for tracking TVL
281                 queuedShares[address(tokens[i])] -= withdrawal.shares[i];
282     
283                 // check if token is not Native ETH
284                 if (address(tokens[i]) != IS_NATIVE) {
285                     // Check the withdraw buffer and fill if below buffer target
286                     uint256 bufferToFill = withdrawQueue.getBufferDeficit(address(tokens[i]));
287     
288                     // get balance of this contract
289                     uint256 balanceOfToken = tokens[i].balanceOf(address(this));
290                     if (bufferToFill > 0) {
291                         bufferToFill = (balanceOfToken <= bufferToFill) ? balanceOfToken : bufferToFill;
292     
293                         // update amount to send to the operator Delegator
294                         balanceOfToken -= bufferToFill;
295     
296                         // safe Approve for depositQueue
297                         tokens[i].safeApprove(address(restakeManager.depositQueue()), bufferToFill);
298     
299                         // fill Withdraw Buffer via depositQueue
300                         restakeManager.depositQueue().fillERC20withdrawBuffer(
301                             address(tokens[i]),
302                             bufferToFill
303                         );
304                     }
305     
306                     // Deposit remaining tokens back to eigenLayer
307                     if (balanceOfToken > 0) {
308                         _deposit(tokens[i], balanceOfToken);
309                     }
310                 }
311                 unchecked {
312                     ++i;
313                 }
314             }
315     
316             // emits the Withdraw Completed event with withdrawalRoot
317             emit WithdrawCompleted(
318                 delegationManager.calculateWithdrawalRoot(withdrawal),
319                 withdrawal.strategies,
320                 withdrawal.shares
321             );
322             // record current spent gas
323             _recordGas(gasBefore);
324         }
364         function verifyWithdrawalCredentials(
365             uint64 oracleTimestamp,
366             BeaconChainProofs.StateRootProof calldata stateRootProof,
367             uint40[] calldata validatorIndices,
368             bytes[] calldata withdrawalCredentialProofs,
369             bytes32[][] calldata validatorFields
370         ) external onlyNativeEthRestakeAdmin {
371             uint256 gasBefore = gasleft();
372             eigenPod.verifyWithdrawalCredentials(
373                 oracleTimestamp,
374                 stateRootProof,
375                 validatorIndices,
376                 withdrawalCredentialProofs,
377                 validatorFields
378             );
379     
380             // Decrement the staked but not verified ETH
381             for (uint256 i = 0; i < validatorFields.length; ) {
382                 uint64 validatorCurrentBalanceGwei = BeaconChainProofs.getEffectiveBalanceGwei(
383                     validatorFields[i]
384                 );
385                 stakedButNotVerifiedEth -= (validatorCurrentBalanceGwei * GWEI_TO_WEI);
386                 unchecked {
387                     ++i;
388                 }
389             }
390             // update the gas spent for RestakeAdmin
391             _recordGas(gasBefore);
392         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


211         function stakeEthFromQueueMulti(
212             IOperatorDelegator[] calldata operatorDelegators,
213             bytes[] calldata pubkeys,
214             bytes[] calldata signatures,
215             bytes32[] calldata depositDataRoots
216         ) external onlyNativeEthRestakeAdmin nonReentrant {
217             uint256 gasBefore = gasleft();
218             // Verify all arrays are the same length
219             if (
220                 operatorDelegators.length != pubkeys.length ||
221                 operatorDelegators.length != signatures.length ||
222                 operatorDelegators.length != depositDataRoots.length
223             ) revert MismatchedArrayLengths();
224     
225             // Iterate through the arrays and stake each one
226             uint256 arrayLength = operatorDelegators.length;
227             for (uint256 i = 0; i < arrayLength; ) {
228                 // Send the ETH and the params through to the restake manager
229                 restakeManager.stakeEthInOperatorDelegator{ value: 32 ether }(
230                     operatorDelegators[i],
231                     pubkeys[i],
232                     signatures[i],
233                     depositDataRoots[i]
234                 );
235     
236                 emit ETHStakedFromQueue(
237                     operatorDelegators[i],
238                     pubkeys[i],
239                     32 ether,
240                     address(this).balance
241                 );
242     
243                 unchecked {
244                     ++i;
245                 }
246             }
247     
248             // Refund the gas to the Admin address if enough ETH available
249             _refundGas(gasBefore);
250         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


131         function addOperatorDelegator(
132             IOperatorDelegator _newOperatorDelegator,
133             uint256 _allocationBasisPoints
134         ) external onlyRestakeManagerAdmin {
135             // Ensure it is not already in the list
136             uint256 odLength = operatorDelegators.length;
137             for (uint256 i = 0; i < odLength; ) {
138                 if (address(operatorDelegators[i]) == address(_newOperatorDelegator))
139                     revert AlreadyAdded();
140                 unchecked {
141                     ++i;
142                 }
143             }
144     
145             // Verify a valid allocation
146             if (_allocationBasisPoints > (100 * BASIS_POINTS)) revert OverMaxBasisPoints();
147     
148             // Add it to the list
149             operatorDelegators.push(_newOperatorDelegator);
150     
151             emit OperatorDelegatorAdded(_newOperatorDelegator);
152     
153             // Set the allocation
154             operatorDelegatorAllocations[_newOperatorDelegator] = _allocationBasisPoints;
155     
156             emit OperatorDelegatorAllocationUpdated(_newOperatorDelegator, _allocationBasisPoints);
157         }
160         function removeOperatorDelegator(
161             IOperatorDelegator _operatorDelegatorToRemove
162         ) external onlyRestakeManagerAdmin {
163             // Remove it from the list
164             uint256 odLength = operatorDelegators.length;
165             for (uint256 i = 0; i < odLength; ) {
166                 if (address(operatorDelegators[i]) == address(_operatorDelegatorToRemove)) {
167                     // Clear the allocation
168                     operatorDelegatorAllocations[_operatorDelegatorToRemove] = 0;
169                     emit OperatorDelegatorAllocationUpdated(_operatorDelegatorToRemove, 0);
170     
171                     // Remove from list
172                     operatorDelegators[i] = operatorDelegators[operatorDelegators.length - 1];
173                     operatorDelegators.pop();
174                     emit OperatorDelegatorRemoved(_operatorDelegatorToRemove);
175                     return;
176                 }
177                 unchecked {
178                     ++i;
179                 }
180             }
181     
182             // If the item was not found, throw an error
183             revert NotFound();
184         }
187         function setOperatorDelegatorAllocation(
188             IOperatorDelegator _operatorDelegator,
189             uint256 _allocationBasisPoints
190         ) external onlyRestakeManagerAdmin {
191             if (address(_operatorDelegator) == address(0x0)) revert InvalidZeroInput();
192             if (_allocationBasisPoints > (100 * BASIS_POINTS)) revert OverMaxBasisPoints();
193     
194             // Ensure the OD is in the list to prevent mis-configuration
195             bool foundOd = false;
196             uint256 odLength = operatorDelegators.length;
197             for (uint256 i = 0; i < odLength; ) {
198                 if (address(operatorDelegators[i]) == address(_operatorDelegator)) {
199                     foundOd = true;
200                     break;
201                 }
202                 unchecked {
203                     ++i;
204                 }
205             }
206             if (!foundOd) revert NotFound();
207     
208             // Set the allocation
209             operatorDelegatorAllocations[_operatorDelegator] = _allocationBasisPoints;
210     
211             emit OperatorDelegatorAllocationUpdated(_operatorDelegator, _allocationBasisPoints);
212         }
220         function addCollateralToken(IERC20 _newCollateralToken) external onlyRestakeManagerAdmin {
221             // Ensure it is not already in the list
222             uint256 tokenLength = collateralTokens.length;
223             for (uint256 i = 0; i < tokenLength; ) {
224                 if (address(collateralTokens[i]) == address(_newCollateralToken)) revert AlreadyAdded();
225                 unchecked {
226                     ++i;
227                 }
228             }
229     
230             // Verify the token has 18 decimal precision - pricing calculations will be off otherwise
231             if (IERC20Metadata(address(_newCollateralToken)).decimals() != 18)
232                 revert InvalidTokenDecimals(
233                     18,
234                     IERC20Metadata(address(_newCollateralToken)).decimals()
235                 );
236     
237             // Add it to the list
238             collateralTokens.push(_newCollateralToken);
239     
240             emit CollateralTokenAdded(_newCollateralToken);
241         }
244         function removeCollateralToken(
245             IERC20 _collateralTokenToRemove
246         ) external onlyRestakeManagerAdmin {
247             // Remove it from the list
248             uint256 tokenLength = collateralTokens.length;
249             for (uint256 i = 0; i < tokenLength; ) {
250                 if (address(collateralTokens[i]) == address(_collateralTokenToRemove)) {
251                     collateralTokens[i] = collateralTokens[collateralTokens.length - 1];
252                     collateralTokens.pop();
253                     emit CollateralTokenRemoved(_collateralTokenToRemove);
254                     return;
255                 }
256                 unchecked {
257                     ++i;
258                 }
259             }
260     
261             // If the item was not found, throw an error
262             revert NotFound();
263         }
491         function deposit(
492             IERC20 _collateralToken,
493             uint256 _amount,
494             uint256 _referralId
495         ) public nonReentrant notPaused {
496             // Verify collateral token is in the list - call will revert if not found
497             uint256 tokenIndex = getCollateralTokenIndex(_collateralToken);
498     
499             // Get the TVLs for each operator delegator and the total TVL
500             (
501                 uint256[][] memory operatorDelegatorTokenTVLs,
502                 uint256[] memory operatorDelegatorTVLs,
503                 uint256 totalTVL
504             ) = calculateTVLs();
505     
506             // Get the value of the collateral token being deposited
507             uint256 collateralTokenValue = renzoOracle.lookupTokenValue(_collateralToken, _amount);
508     
509             // Enforce TVL limit if set, 0 means the check is not enabled
510             if (maxDepositTVL != 0 && totalTVL + collateralTokenValue > maxDepositTVL) {
511                 revert MaxTVLReached();
512             }
513     
514             // Enforce individual token TVL limit if set, 0 means the check is not enabled
515             if (collateralTokenTvlLimits[_collateralToken] != 0) {
516                 // Track the current token's TVL
517                 uint256 currentTokenTVL = 0;
518     
519                 // For each OD, add up the token TVLs
520                 uint256 odLength = operatorDelegatorTokenTVLs.length;
521                 for (uint256 i = 0; i < odLength; ) {
522                     currentTokenTVL += operatorDelegatorTokenTVLs[i][tokenIndex];
523                     unchecked {
524                         ++i;
525                     }
526                 }
527     
528                 // Check if it is over the limit
529                 if (currentTokenTVL + collateralTokenValue > collateralTokenTvlLimits[_collateralToken])
530                     revert MaxTokenTVLReached();
531             }
532     
533             // Determine which operator delegator to use
534             IOperatorDelegator operatorDelegator = chooseOperatorDelegatorForDeposit(
535                 operatorDelegatorTVLs,
536                 totalTVL
537             );
538     
539             // Transfer the collateral token to this address
540             _collateralToken.safeTransferFrom(msg.sender, address(this), _amount);
541     
542             // Check the withdraw buffer and fill if below buffer target
543             uint256 bufferToFill = depositQueue.withdrawQueue().getBufferDeficit(
544                 address(_collateralToken)
545             );
546             if (bufferToFill > 0) {
547                 bufferToFill = (_amount <= bufferToFill) ? _amount : bufferToFill;
548                 // update amount to send to the operator Delegator
549                 _amount -= bufferToFill;
550     
551                 // safe Approve for depositQueue
552                 _collateralToken.safeApprove(address(depositQueue), bufferToFill);
553     
554                 // fill Withdraw Buffer via depositQueue
555                 depositQueue.fillERC20withdrawBuffer(address(_collateralToken), bufferToFill);
556             }
557     
558             // Approve the tokens to the operator delegator
559             _collateralToken.safeApprove(address(operatorDelegator), _amount);
560     
561             // Call deposit on the operator delegator
562             operatorDelegator.deposit(_collateralToken, _amount);
563     
564             // Calculate how much ezETH to mint
565             uint256 ezETHToMint = renzoOracle.calculateMintAmount(
566                 totalTVL,
567                 collateralTokenValue,
568                 ezETH.totalSupply()
569             );
570     
571             // Mint the ezETH
572             ezETH.mint(msg.sender, ezETHToMint);
573     
574             // Emit the deposit event
575             emit Deposit(msg.sender, _collateralToken, _amount, ezETHToMint, _referralId);
576         }
620         function stakeEthInOperatorDelegator(
621             IOperatorDelegator operatorDelegator,
622             bytes calldata pubkey,
623             bytes calldata signature,
624             bytes32 depositDataRoot
625         ) external payable onlyDepositQueue {
626             // Verify the OD is in the list
627             bool found = false;
628             uint256 odLength = operatorDelegators.length;
629             for (uint256 i = 0; i < odLength; ) {
630                 if (operatorDelegators[i] == operatorDelegator) {
631                     found = true;
632                     break;
633                 }
634     
635                 unchecked {
636                     ++i;
637                 }
638             }
639             if (!found) revert NotFound();
640     
641             // Call the OD to stake the ETH
642             operatorDelegator.stakeEth{ value: msg.value }(pubkey, signature, depositDataRoot);
643         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/TimelockController.sol


259         function scheduleBatch(
260             address[] calldata targets,
261             uint256[] calldata values,
262             bytes[] calldata payloads,
263             bytes32 predecessor,
264             bytes32 salt,
265             uint256 delay
266         ) public virtual onlyRole(PROPOSER_ROLE) {
267             require(targets.length == values.length, "TimelockController: length mismatch");
268             require(targets.length == payloads.length, "TimelockController: length mismatch");
269     
270             bytes32 id = hashOperationBatch(targets, values, payloads, predecessor, salt);
271             _schedule(id, delay);
272             for (uint256 i = 0; i < targets.length; ++i) {
273                 emit CallScheduled(id, i, targets[i], values[i], payloads[i], predecessor, delay);
274             }
275             if (salt != bytes32(0)) {
276                 emit CallSalt(id, salt);
277             }
278         }
342         function executeBatch(
343             address[] calldata targets,
344             uint256[] calldata values,
345             bytes[] calldata payloads,
346             bytes32 predecessor,
347             bytes32 salt
348         ) public payable virtual onlyRoleOrOpenRole(EXECUTOR_ROLE) {
349             require(targets.length == values.length, "TimelockController: length mismatch");
350             require(targets.length == payloads.length, "TimelockController: length mismatch");
351     
352             bytes32 id = hashOperationBatch(targets, values, payloads, predecessor, salt);
353     
354             _beforeCall(id, predecessor);
355             for (uint256 i = 0; i < targets.length; ++i) {
356                 address target = targets[i];
357                 uint256 value = values[i];
358                 bytes calldata payload = payloads[i];
359                 _execute(target, value, payload);
360                 emit CallExecuted(id, i, target, value, payload);
361             }
362             _afterCall(id);
363         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


64          function initialize(
65              IRoleManager _roleManager,
66              IRestakeManager _restakeManager,
67              IEzEthToken _ezETH,
68              IRenzoOracle _renzoOracle,
69              uint256 _coolDownPeriod,
70              TokenWithdrawBuffer[] calldata _withdrawalBufferTarget
71          ) external initializer {
72              if (
73                  address(_roleManager) == address(0) ||
74                  address(_ezETH) == address(0) ||
75                  address(_renzoOracle) == address(0) ||
76                  address(_restakeManager) == address(0) ||
77                  _withdrawalBufferTarget.length == 0 ||
78                  _coolDownPeriod == 0
79              ) revert InvalidZeroInput();
80      
81              __Pausable_init();
82      
83              roleManager = _roleManager;
84              restakeManager = _restakeManager;
85              ezETH = _ezETH;
86              renzoOracle = _renzoOracle;
87              coolDownPeriod = _coolDownPeriod;
88              for (uint256 i = 0; i < _withdrawalBufferTarget.length; ) {
89                  if (
90                      _withdrawalBufferTarget[i].asset == address(0) ||
91                      _withdrawalBufferTarget[i].bufferAmount == 0
92                  ) revert InvalidZeroInput();
93                  withdrawalBufferTarget[_withdrawalBufferTarget[i].asset] = _withdrawalBufferTarget[i]
94                      .bufferAmount;
95                  unchecked {
96                      ++i;
97                  }
98              }
99          }
106         function updateWithdrawBufferTarget(
107             TokenWithdrawBuffer[] calldata _newBufferTarget
108         ) external onlyWithdrawQueueAdmin {
109             if (_newBufferTarget.length == 0) revert InvalidZeroInput();
110             for (uint256 i = 0; i < _newBufferTarget.length; ) {
111                 if (_newBufferTarget[i].asset == address(0) || _newBufferTarget[i].bufferAmount == 0)
112                     revert InvalidZeroInput();
113                 emit WithdrawBufferTargetUpdated(
114                     withdrawalBufferTarget[_newBufferTarget[i].asset],
115                     _newBufferTarget[i].bufferAmount
116                 );
117                 withdrawalBufferTarget[_newBufferTarget[i].asset] = _newBufferTarget[i].bufferAmount;
118                 unchecked {
119                     ++i;
120                 }
121             }
122         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## L026 - Function calls within for loops:

Making function calls within loops in Solidity can lead to inefficient gas usage, potential bottlenecks, and increased vulnerability to attacks. Each function call or external call consumes gas, and when executed within a loop, the gas cost multiplies, potentially causing the transaction to run out of gas or exceed block gas limits. This can result in transaction failure or unpredictable behavior.


```solidity
File: contracts/Delegation/OperatorDelegator.sol


277             for (uint256 i; i < tokens.length; ) {
278                 if (address(tokens[i]) == address(0)) revert InvalidZeroInput();
279     
280                 // deduct queued shares for tracking TVL
281                 queuedShares[address(tokens[i])] -= withdrawal.shares[i];
282     
283                 // check if token is not Native ETH
284                 if (address(tokens[i]) != IS_NATIVE) {
285                     // Check the withdraw buffer and fill if below buffer target
286                     uint256 bufferToFill = withdrawQueue.getBufferDeficit(address(tokens[i]));
287     
288                     // get balance of this contract
289                     uint256 balanceOfToken = tokens[i].balanceOf(address(this));
290                     if (bufferToFill > 0) {
291                         bufferToFill = (balanceOfToken <= bufferToFill) ? balanceOfToken : bufferToFill;
292     
293                         // update amount to send to the operator Delegator
294                         balanceOfToken -= bufferToFill;
295     
296                         // safe Approve for depositQueue
297                         tokens[i].safeApprove(address(restakeManager.depositQueue()), bufferToFill);
298     
299                         // fill Withdraw Buffer via depositQueue
300                         restakeManager.depositQueue().fillERC20withdrawBuffer(
301                             address(tokens[i]),
302                             bufferToFill
303                         );
304                     }
305     
306                     // Deposit remaining tokens back to eigenLayer
307                     if (balanceOfToken > 0) {
308                         _deposit(tokens[i], balanceOfToken);
309                     }
310                 }
311                 unchecked {
312                     ++i;
313                 }
314             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


111             for (uint256 i = 0; i < tokenLength; ) {
112                 totalValue += lookupTokenValue(_tokens[i], _balances[i]);
113                 unchecked {
114                     ++i;
115                 }
116             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/TimelockController.sol


355             for (uint256 i = 0; i < targets.length; ++i) {
356                 address target = targets[i];
357                 uint256 value = values[i];
358                 bytes calldata payload = payloads[i];
359                 _execute(target, value, payload);
360                 emit CallExecuted(id, i, target, value, payload);
361             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

## L027 - External calls in an unbounded for-loop may result in a DoS:

Consider limiting the number of iterations in for-loops that make external calls.


<details>
<summary>Click to show 4 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


218             for (uint256 i = 0; i < _destinationParam.length; ) {
264             for (uint256 i = 0; i < _connextDestinationParam.length; ) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


209             for (uint256 i; i < tokens.length; ) {
277             for (uint256 i; i < tokens.length; ) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/TimelockController.sol


107             for (uint256 i = 0; i < proposers.length; ++i) {
113             for (uint256 i = 0; i < executors.length; ++i) {
272             for (uint256 i = 0; i < targets.length; ++i) {
355             for (uint256 i = 0; i < targets.length; ++i) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


88              for (uint256 i = 0; i < _withdrawalBufferTarget.length; ) {
110             for (uint256 i = 0; i < _newBufferTarget.length; ) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## L028 - Contracts are not using their OZ Upgradeable counterparts:

The non-upgradeable standard version of OpenZeppelin’s library is inherited/used by the contracts. It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behavior.

Use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts` where applicable. See https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/tree/master/contracts for a list of available upgradeable contracts


<details>
<summary>Click to show 27 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


4       import { SafeERC20 } from "contest/node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
5       import { IERC20 } from "contest/node_modules/@openzeppelin/contracts/token/ERC20/IERC20.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
7       import {
14      import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridgeStorage.sol


5       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridgeStorage.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


6       import {
5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


8       import { Ownable } from "contest/node_modules/@openzeppelin/contracts/access/Ownable.sol";
9       import { Pausable } from "contest/node_modules/@openzeppelin/contracts/security/Pausable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


6       import { Pausable } from "contest/node_modules/@openzeppelin/contracts/security/Pausable.sol";
5       import { Ownable } from "contest/node_modules/@openzeppelin/contracts/access/Ownable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


6       import {
5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
15      import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
9       import {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


5       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


8       import {
5       import {
11      import {
14      import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


8       import {
12      import "contest/node_modules/@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";
11      import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


6       import { SafeERC20 } from "contest/node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
5       import { IERC20 } from "contest/node_modules/@openzeppelin/contracts/token/ERC20/ERC20.sol";
7       import { SafeCast } from "contest/node_modules/@openzeppelin/contracts/utils/math/SafeCast.sol";
9       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


4       import {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


8       import {
12      import "contest/node_modules/@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";
11      import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


4       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
5       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
7       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracleStorage.sol


6       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/IERC20.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracleStorage.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProviderStorage.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProviderStorage.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


4       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
8       import "contest/node_modules/@openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";
5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
7       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
9       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/IERC20.sol";
10      import "contest/node_modules/@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
6       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
6       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


6       import "contest/node_modules/@openzeppelin/contracts/access/AccessControl.sol";
8       import "contest/node_modules/@openzeppelin/contracts/token/ERC1155/IERC1155Receiver.sol";
7       import "contest/node_modules/@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


6       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
8       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
7       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";
9       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/IERC20.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## L029 - Consider implementing two-step procedure for updating protocol addresses:

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions. See similar findings in previous Code4rena contests for reference: https://code4rena.com/reports/2022-06-illuminate/#2-critical-changes-should-use-two-step-procedure


```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


134         function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


130         function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0


## L030 - Missing checks for address(0x0) in the initializer:

  


<details>
<summary>Click to show 7 findings</summary>

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


75          function initialize(
76              uint256 _currentPrice,
77              IERC20 _xezETH,
78              IERC20 _depositToken,
79              IERC20 _collateralToken,
80              IConnext _connext,
81              bytes32 _swapKey,
82              address _receiver,
83              uint32 _bridgeDestinationDomain,
84              address _bridgeTargetAddress,
85              IRenzoOracleL2 _oracle
86          ) public initializer {
87              // Initialize inherited classes
88              __Ownable_init();
89      
90              // Verify valid non zero values
91              if (
92                  _currentPrice == 0 ||
93                  address(_xezETH) == address(0) ||
94                  address(_depositToken) == address(0) ||
95                  address(_collateralToken) == address(0) ||
96                  address(_connext) == address(0) ||
97                  _swapKey == 0 ||
98                  _bridgeDestinationDomain == 0 ||
99                  _bridgeTargetAddress == address(0)
100             ) {
101                 revert InvalidZeroInput();
102             }
103     
104             // Verify all tokens have 18 decimals
105             uint8 decimals = IERC20MetadataUpgradeable(address(_depositToken)).decimals();
106             if (decimals != EXPECTED_DECIMALS) {
107                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
108             }
109             decimals = IERC20MetadataUpgradeable(address(_collateralToken)).decimals();
110             if (decimals != EXPECTED_DECIMALS) {
111                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
112             }
113             decimals = IERC20MetadataUpgradeable(address(_xezETH)).decimals();
114             if (decimals != EXPECTED_DECIMALS) {
115                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
116             }
117     
118             // Initialize the price and timestamp
119             lastPrice = _currentPrice;
120             lastPriceTimestamp = block.timestamp;
121     
122             // Set xezETH address
123             xezETH = _xezETH;
124     
125             // Set the depoist token
126             depositToken = _depositToken;
127     
128             // Set the collateral token
129             collateralToken = _collateralToken;
130     
131             // Set the connext contract
132             connext = _connext;
133     
134             // Set the swap key
135             swapKey = _swapKey;
136     
137             // Set receiver contract address
138             receiver = _receiver;
139             // Connext router fee is 5 basis points
140             bridgeRouterFeeBps = 5;
141     
142             // Set the bridge destination domain
143             bridgeDestinationDomain = _bridgeDestinationDomain;
144     
145             // Set the bridge target address
146             bridgeTargetAddress = _bridgeTargetAddress;
147     
148             // set oracle Price Feed struct
149             oracle = _oracle;
150     
151             // set bridge Fee Share 0.05% where 100 basis point = 1%
152             bridgeFeeShare = 5;
153     
154             //set sweep batch size to 32 ETH
155             sweepBatchSize = 32 ether;
156         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


61          function initialize(
62              string memory _name,
63              string memory _symbol,
64              address _factory
65          ) public initializer {
66              __XERC20_init(_name, _symbol, _factory);
67          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


54          function initialize(
55              address _lockboxImplementation,
56              address _xerc20Implementation
57          ) public initializer {
58              lockboxImplementation = _lockboxImplementation;
59              xerc20Implementation = _xerc20Implementation;
60          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


44          function initialize(address _xerc20, address _erc20, bool _isNative) public initializer {
45              XERC20 = IXERC20(_xerc20);
46              ERC20 = IERC20(_erc20);
47              IS_NATIVE = _isNative;
48          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


35          function initialize(
36              string memory _name,
37              string memory _symbol,
38              address _factory,
39              address _l1Token,
40              address _optimismBridge
41          ) public initializer {
42              __ERC165_init();
43              __XERC20_init(_name, _symbol, _factory);
44              l1Token = _l1Token;
45              optimismBridge = _optimismBridge;
46          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


22          function initialize(address roleManagerAdmin) public initializer {
23              if (address(roleManagerAdmin) == address(0x0)) revert InvalidZeroInput();
24      
25              __AccessControl_init();
26      
27              _grantRole(DEFAULT_ADMIN_ROLE, roleManagerAdmin);
28          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


38          function initialize(IRoleManager _roleManager, address _rewardDestination) public initializer {
39              __ReentrancyGuard_init();
40      
41              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
42              if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();
43      
44              roleManager = _roleManager;
45              rewardDestination = _rewardDestination;
46      
47              emit RewardDestinationUpdated(_rewardDestination);
48          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

</details>










## NC001 - The `nonReentrant` `modifier` should occur before all other modifiers:

This is a best-practice to protect against reentrancy in other modifiers.


```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


210         function sendPrice(
211             CCIPDestinationParam[] calldata _destinationParam,
212             ConnextDestinationParam[] calldata _connextDestinationParam
213         ) external payable onlyPriceFeedSender nonReentrant {
214             // call getRate() to get the current price of ezETH
215             uint256 exchangeRate = rateProvider.getRate();
216             bytes memory _callData = abi.encode(exchangeRate, block.timestamp);
217             // send price feed to renzo CCIP receivers
218             for (uint256 i = 0; i < _destinationParam.length; ) {
219                 Client.EVM2AnyMessage memory evm2AnyMessage = Client.EVM2AnyMessage({
220                     receiver: abi.encode(_destinationParam[i]._renzoReceiver), // ABI-encoded xRenzoDepsot contract address
221                     data: _callData, // ABI-encoded ezETH exchange rate with Timestamp
222                     tokenAmounts: new Client.EVMTokenAmount[](0), // Empty array indicating no tokens are being sent
223                     extraArgs: Client._argsToBytes(
224                         // Additional arguments, setting gas limit
225                         Client.EVMExtraArgsV1({ gasLimit: 200_000 })
226                     ),
227                     // Set the feeToken  address, indicating LINK will be used for fees
228                     feeToken: address(linkToken)
229                 });
230     
231                 // Get the fee required to send the message
232                 uint256 fees = linkRouterClient.getFee(
233                     _destinationParam[i].destinationChainSelector,
234                     evm2AnyMessage
235                 );
236     
237                 if (fees > linkToken.balanceOf(address(this)))
238                     revert NotEnoughBalance(linkToken.balanceOf(address(this)), fees);
239     
240                 // approve the Router to transfer LINK tokens on contract's behalf. It will spend the fees in LINK
241                 linkToken.approve(address(linkRouterClient), fees);
242     
243                 // Send the message through the router and store the returned message ID
244                 bytes32 messageId = linkRouterClient.ccipSend(
245                     _destinationParam[i].destinationChainSelector,
246                     evm2AnyMessage
247                 );
248     
249                 // Emit an event with message details
250                 emit MessageSent(
251                     messageId,
252                     _destinationParam[i].destinationChainSelector,
253                     _destinationParam[i]._renzoReceiver,
254                     exchangeRate,
255                     address(linkToken),
256                     fees
257                 );
258                 unchecked {
259                     ++i;
260                 }
261             }
262     
263             // send price feed to renzo connext receiver
264             for (uint256 i = 0; i < _connextDestinationParam.length; ) {
265                 connext.xcall{ value: _connextDestinationParam[i].relayerFee }(
266                     _connextDestinationParam[i].destinationDomainId,
267                     _connextDestinationParam[i]._renzoReceiver,
268                     address(0),
269                     msg.sender,
270                     0,
271                     0,
272                     _callData
273                 );
274     
275                 emit ConnextMessageSent(
276                     _connextDestinationParam[i].destinationDomainId,
277                     _connextDestinationParam[i]._renzoReceiver,
278                     exchangeRate,
279                     _connextDestinationParam[i].relayerFee
280                 );
281     
282                 unchecked {
283                     ++i;
284                 }
285             }
286         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


211         function stakeEthFromQueueMulti(
212             IOperatorDelegator[] calldata operatorDelegators,
213             bytes[] calldata pubkeys,
214             bytes[] calldata signatures,
215             bytes32[] calldata depositDataRoots
216         ) external onlyNativeEthRestakeAdmin nonReentrant {
217             uint256 gasBefore = gasleft();
218             // Verify all arrays are the same length
219             if (
220                 operatorDelegators.length != pubkeys.length ||
221                 operatorDelegators.length != signatures.length ||
222                 operatorDelegators.length != depositDataRoots.length
223             ) revert MismatchedArrayLengths();
224     
225             // Iterate through the arrays and stake each one
226             uint256 arrayLength = operatorDelegators.length;
227             for (uint256 i = 0; i < arrayLength; ) {
228                 // Send the ETH and the params through to the restake manager
229                 restakeManager.stakeEthInOperatorDelegator{ value: 32 ether }(
230                     operatorDelegators[i],
231                     pubkeys[i],
232                     signatures[i],
233                     depositDataRoots[i]
234                 );
235     
236                 emit ETHStakedFromQueue(
237                     operatorDelegators[i],
238                     pubkeys[i],
239                     32 ether,
240                     address(this).balance
241                 );
242     
243                 unchecked {
244                     ++i;
245                 }
246             }
247     
248             // Refund the gas to the Admin address if enough ETH available
249             _refundGas(gasBefore);
250         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0


## NC002 - Defining All External/Public Functions in Contract Interfaces:

It is preferable to have all the external and public function in an interface to make using them easier by developers. This helps ensure the whole API is extracted in a interface.


<details>
<summary>Click to show 20 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


70          function initialize(
71              IERC20 _ezETH,
72              IERC20 _xezETH,
73              IRestakeManager _restakeManager,
74              IERC20 _wETH,
75              IXERC20Lockbox _xezETHLockbox,
76              IConnext _connext,
77              IRouterClient _linkRouterClient,
78              IRateProvider _rateProvider,
79              LinkTokenInterface _linkToken,
80              IRoleManager _roleManager
81          ) public initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


23          function initialize(AggregatorV3Interface _oracle) public initializer {
50          function getMintRate() public view returns (uint256, uint256) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


75          function initialize(
76              uint256 _currentPrice,
77              IERC20 _xezETH,
78              IERC20 _depositToken,
79              IERC20 _collateralToken,
80              IConnext _connext,
81              bytes32 _swapKey,
82              address _receiver,
83              uint32 _bridgeDestinationDomain,
84              address _bridgeTargetAddress,
85              IRenzoOracleL2 _oracle
86          ) public initializer {
277         function getBridgeFeeShare(uint256 _amountIn) public view returns (uint256) {
289         function getMintRate() public view returns (uint256, uint256) {
414         function sweep() public payable nonReentrant {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


61          function initialize(
62              string memory _name,
63              string memory _symbol,
64              address _factory
65          ) public initializer {
96          function mint(address _user, uint256 _amount) public virtual {
107         function burn(address _user, uint256 _amount) public virtual {
121         function setLockbox(address _lockbox) public {
152         function mintingMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
163         function burningMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
174         function mintingCurrentLimitOf(address _bridge) public view returns (uint256 _limit) {
190         function burningCurrentLimitOf(address _bridge) public view returns (uint256 _limit) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


54          function initialize(
55              address _lockboxImplementation,
56              address _xerc20Implementation
57          ) public initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


44          function initialize(address _xerc20, address _erc20, bool _isNative) public initializer {
54          function depositNative() public payable {
91          function depositNativeTo(address _to) public payable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


35          function initialize(
36              string memory _name,
37              string memory _symbol,
38              address _factory,
39              address _l1Token,
40              address _optimismBridge
41          ) public initializer {
48          function supportsInterface(
49              bytes4 interfaceId
50          ) public view override(ERC165Upgradeable) returns (bool) {
56          function remoteToken() public view override returns (address) {
60          function bridge() public view override returns (address) {
64          function mint(address _to, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
68          function burn(address _from, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


37          function deployOptimismMintableXERC20(
38              string memory _name,
39              string memory _symbol,
40              uint256[] memory _minterLimits,
41              uint256[] memory _burnerLimits,
42              address[] memory _bridges,
43              address _proxyAdmin,
44              address _l1Token
45          ) public returns (address _xerc20) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


172         function getStrategyIndex(IStrategy _strategy) public view returns (uint256) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


74          function initialize(IRoleManager _roleManager) public initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


23          function initialize(IStakedTokenV2 _wBETHToken) public initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


23          function initialize(IMethStaking _methStaking) public initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


44          function initialize(IRoleManager _roleManager) public initializer {
71          function lookupTokenValue(IERC20 _token, uint256 _balance) public view returns (uint256) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


22          function initialize(address roleManagerAdmin) public initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


17          function initialize(
18              IRestakeManager _restakeManager,
19              IERC20Upgradeable _ezETHToken
20          ) public initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


101         function initialize(
102             IRoleManager _roleManager,
103             IEzEthToken _ezETH,
104             IRenzoOracle _renzoOracle,
105             IStrategyManager _strategyManager,
106             IDelegationManager _delegationManager,
107             IDepositQueue _depositQueue
108         ) public initializer {
274         function calculateTVLs() public view returns (uint256[][] memory, uint256[] memory, uint256) {
362         function chooseOperatorDelegatorForDeposit(
363             uint256[] memory tvls,
364             uint256 totalTVL
365         ) public view returns (IOperatorDelegator) {
400         function chooseOperatorDelegatorForWithdraw(
401             uint256 tokenIndex,
402             uint256 ezETHValue,
403             uint256[][] memory operatorDelegatorTokenTVLs,
404             uint256[] memory operatorDelegatorTVLs,
405             uint256 totalTVL
406         ) public view returns (IOperatorDelegator) {
451         function getCollateralTokenIndex(IERC20 _collateralToken) public view returns (uint256) {
491         function deposit(
492             IERC20 _collateralToken,
493             uint256 _amount,
494             uint256 _referralId
495         ) public nonReentrant notPaused {
592         function depositETH(uint256 _referralId) public payable nonReentrant notPaused {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


38          function initialize(IRoleManager _roleManager, address _rewardDestination) public initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


142         function supportsInterface(
143             bytes4 interfaceId
144         ) public view virtual override(IERC165, AccessControl) returns (bool) {
154         function isOperation(bytes32 id) public view virtual returns (bool) {
161         function isOperationPending(bytes32 id) public view virtual returns (bool) {
168         function isOperationReady(bytes32 id) public view virtual returns (bool) {
176         function isOperationDone(bytes32 id) public view virtual returns (bool) {
184         function getTimestamp(bytes32 id) public view virtual returns (uint256) {
193         function getMinDelay() public view virtual returns (uint256) {
201         function hashOperation(
202             address target,
203             uint256 value,
204             bytes calldata data,
205             bytes32 predecessor,
206             bytes32 salt
207         ) public pure virtual returns (bytes32) {
215         function hashOperationBatch(
216             address[] calldata targets,
217             uint256[] calldata values,
218             bytes[] calldata payloads,
219             bytes32 predecessor,
220             bytes32 salt
221         ) public pure virtual returns (bytes32) {
234         function schedule(
235             address target,
236             uint256 value,
237             bytes calldata data,
238             bytes32 predecessor,
239             bytes32 salt,
240             uint256 delay
241         ) public virtual onlyRole(PROPOSER_ROLE) {
259         function scheduleBatch(
260             address[] calldata targets,
261             uint256[] calldata values,
262             bytes[] calldata payloads,
263             bytes32 predecessor,
264             bytes32 salt,
265             uint256 delay
266         ) public virtual onlyRole(PROPOSER_ROLE) {
296         function cancel(bytes32 id) public virtual onlyRole(CANCELLER_ROLE) {
315         function execute(
316             address target,
317             uint256 value,
318             bytes calldata payload,
319             bytes32 predecessor,
320             bytes32 salt
321         ) public payable virtual onlyRoleOrOpenRole(EXECUTOR_ROLE) {
342         function executeBatch(
343             address[] calldata targets,
344             uint256[] calldata values,
345             bytes[] calldata payloads,
346             bytes32 predecessor,
347             bytes32 salt
348         ) public payable virtual onlyRoleOrOpenRole(EXECUTOR_ROLE) {
411         function onERC721Received(
412             address,
413             address,
414             uint256,
415             bytes memory
416         ) public virtual override returns (bytes4) {
423         function onERC1155Received(
424             address,
425             address,
426             uint256,
427             uint256,
428             bytes memory
429         ) public virtual override returns (bytes4) {
436         function onERC1155BatchReceived(
437             address,
438             address,
439             uint256[] memory,
440             uint256[] memory,
441             bytes memory
442         ) public virtual override returns (bytes4) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


156         function getAvailableToWithdraw(address _asset) public view returns (uint256) {
170         function getBufferDeficit(address _asset) public view returns (uint256) {
270         function getOutstandingWithdrawRequests(address user) public view returns (uint256) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


33          function initialize(IRoleManager _roleManager) public initializer {
77          function name() public view virtual override returns (string memory) {
85          function symbol() public view virtual override returns (string memory) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>



## NC003 - Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18):

While the compiler knows to optimize away the exponentiation, it's still better coding practice to use idioms that do not require compiler optimization, if they exist.


```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


54              uint256 _scaledPrice = (uint256(price)) * 10 ** (18 - oracle.decimals());


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


23          uint256 constant SCALE_FACTOR = 10 ** 18;
23          uint256 constant SCALE_FACTOR = 10 ** 18;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


40              return (10 ** 18 * totalTVL) / totalSupply;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0


## NC004 - Empty bytes check is missing:

When developing smart contracts in Solidity, it's crucial to validate the inputs of your functions. This includes ensuring that the bytes parameters are not empty, especially when they represent crucial data such as addresses, identifiers, or raw data that the contract needs to process.Missing empty bytes checks can lead to unexpected behaviour in your contract.For instance, certain operations might fail, produce incorrect results, or consume unnecessary gas when performed with empty bytes.  Moreover, missing input validation can potentially expose your contract to malicious activity, including exploitation of unhandled edge cases.To mitigate these issues, always validate that bytes parameters are not empty when the logic of your contract requires it.


<details>
<summary>Click to show 6 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


56          function bridgeTo(
57              address _to,
58              address _erc20,
59              address _remoteToken,
60              uint256 _amount,
61              uint32 _minGasLimit,
62              bytes calldata _extraData
63          ) external {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


69          function xReceive(
70              bytes32 _transferId,
71              uint256,
72              address,
73              address _originSender,
74              uint32 _origin,
75              bytes memory _callData
76          ) external onlySource(_originSender, _origin) whenNotPaused returns (bytes memory) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


349         function stakeEth(
350             bytes calldata pubkey,
351             bytes calldata signature,
352             bytes32 depositDataRoot
353         ) external payable onlyRestakeManager {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


187         function stakeEthFromQueue(
188             IOperatorDelegator operatorDelegator,
189             bytes calldata pubkey,
190             bytes calldata signature,
191             bytes32 depositDataRoot
192         ) external onlyNativeEthRestakeAdmin {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


620         function stakeEthInOperatorDelegator(
621             IOperatorDelegator operatorDelegator,
622             bytes calldata pubkey,
623             bytes calldata signature,
624             bytes32 depositDataRoot
625         ) external payable onlyDepositQueue {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/TimelockController.sol


201         function hashOperation(
202             address target,
203             uint256 value,
204             bytes calldata data,
205             bytes32 predecessor,
206             bytes32 salt
207         ) public pure virtual returns (bytes32) {
234         function schedule(
235             address target,
236             uint256 value,
237             bytes calldata data,
238             bytes32 predecessor,
239             bytes32 salt,
240             uint256 delay
241         ) public virtual onlyRole(PROPOSER_ROLE) {
315         function execute(
316             address target,
317             uint256 value,
318             bytes calldata payload,
319             bytes32 predecessor,
320             bytes32 salt
321         ) public payable virtual onlyRoleOrOpenRole(EXECUTOR_ROLE) {
368         function _execute(address target, uint256 value, bytes calldata data) internal virtual {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

</details>


## NC005 - Avoid defining a function in a single line including it's contents:

  


```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


313         receive() external payable {}


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


542         receive() external payable {}


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/TimelockController.sol


137         receive() external payable {}


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0


## NC006 - Imports could be organized more systematically:

This issue arises when the contract's interface is not imported first, followed by each of the interfaces it uses, followed by all other files.


<details>
<summary>Click to show 17 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


5       import { IERC20 } from "contest/node_modules/@openzeppelin/contracts/token/ERC20/IERC20.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


10      import "../Connext/core/IXReceiver.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridgeStorage.sol


6       import "../../IRestakeManager.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridgeStorage.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


13      import "../xERC20/interfaces/IXERC20.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


6       import "../Connext/core/IConnext.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


8       import { IXERC20Lockbox } from "../interfaces/IXERC20Lockbox.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


6       import "../Permissions/IRoleManager.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShimStorage.sol


5       import "./IStakedTokenV2.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShimStorage.sol


5       import "./IMethStaking.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


5       import "../Permissions/IRoleManager.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracleStorage.sol


6       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/IERC20.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracleStorage.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


5       import "./IRoleManager.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


5       import "./IRateProvider.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


8       import "contest/node_modules/@openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/TimelockController.sol


7       import "contest/node_modules/@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


9       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/IERC20.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


6       import "./IEzEthToken.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## NC007 - Constants in comparisons should appear on the left side:

This issue arises when constants in comparisons appear on the right side, which can lead to typo bugs.


<details>
<summary>Click to show 16 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


65              if (_amount <= 0) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


158             if (_amount == 0) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


30              if (_oracle.decimals() > 18) revert InvalidTokenDecimals(18, _oracle.decimals());
39              if (_oracleAddress.decimals() > 18)
55              if (_scaledPrice < 1 ether) revert InvalidOraclePrice();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


48              if (_xRenzoBridgeL1 == address(0) || _ccipEthChainSelector == 0) revert InvalidZeroInput();
108             if (_newChainSelector == 0) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


53              if (_xRenzoBridgeL1 == address(0) || _connextEthChainDomain == 0 || _connext == address(0))
104             if (_newChainDomain == 0) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


92                  _currentPrice == 0 ||
97                  _swapKey == 0 ||
98                  _bridgeDestinationDomain == 0 ||
172             if (msg.value == 0) {
186             if (wrappedAmount == 0) {
209             if (_amountIn == 0) {
240             if (amountOut == 0) {
332             if (_price == 0) {
385             if (bridgeRouterFeeBps > 0) {
424             if (balance == 0) {
522             if (_newShare > 100) revert InvalidBridgeFeeShare(_newShare);
533             if (_newBatchSize < 32 ether) revert InvalidSweepBatchSize(_newBatchSize);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


330             if (_amount == 0) revert IXERC20_INVALID_0_VALUE();
350             if (_amount == 0) revert IXERC20_INVALID_0_VALUE();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


86              if (_bridgesLength < 1) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


135             if (_baseGasAmountSpent == 0) revert InvalidZeroInput();
147             if (address(tokenStrategyMapping[token]) == address(0x0) || tokenAmount == 0)
290                     if (bufferToFill > 0) {
307                     if (balanceOfToken > 0) {
329                 queuedShares[address(this)] == 0
342                 podOwnerShares < 0
509                 if (adminGasSpentInWei[tx.origin] > 0) {
514                     if (remainingAmount == 0) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


98              if (_feeBasisPoints > 0) {
103             if (_feeBasisPoints > 10000) revert OverMaxBasisPoints();
138             if (_amount == 0 || _asset == address(0)) revert InvalidZeroInput();
166             if (feeAddress != address(0x0) && feeBasisPoints > 0) {
256             if (balance > 0) {
260                 if (feeAddress != address(0x0) && feeBasisPoints > 0) {
298             if (bufferToFill > 0) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


61              if (_oracleAddress.decimals() != 18)
77              if (price <= 0) revert InvalidOraclePrice();
94              if (price <= 0) revert InvalidOraclePrice();
130             if (_currentValueInProtocol == 0 || _existingEzETHSupply == 0) {
146             if (mintAmount == 0) revert InvalidTokenAmount();
161             if (redeemAmount == 0) revert InvalidTokenAmount();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


37              if (totalSupply == 0 || totalTVL == 0) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


231             if (IERC20Metadata(address(_newCollateralToken)).decimals() != 18)
367             if (operatorDelegators.length == 0) revert NotFound();
370             if (operatorDelegators.length == 1) {
408             if (operatorDelegators.length == 1) {
510             if (maxDepositTVL != 0 && totalTVL + collateralTokenValue > maxDepositTVL) {
515             if (collateralTokenTvlLimits[_collateralToken] != 0) {
546             if (bufferToFill > 0) {
597             if (maxDepositTVL != 0 && totalTVL + msg.value > maxDepositTVL) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


64              if (balance == 0) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


155             return getTimestamp(id) > 0;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


77                  _withdrawalBufferTarget.length == 0 ||
78                  _coolDownPeriod == 0
91                      _withdrawalBufferTarget[i].bufferAmount == 0
109             if (_newBufferTarget.length == 0) revert InvalidZeroInput();
111                 if (_newBufferTarget[i].asset == address(0) || _newBufferTarget[i].bufferAmount == 0)
130             if (_newCoolDownPeriod == 0) revert InvalidZeroInput();
196             if (_asset == address(0) || _amount == 0) revert InvalidZeroInput();
208             if (_amount == 0 || _assetOut == address(0)) revert InvalidZeroInput();
211             if (withdrawalBufferTarget[_assetOut] == 0) revert UnsupportedWithdrawAsset();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## NC008 - else-block not required:

One level of nesting can be removed by not having an else block when the if-block returns


```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


279             if (_amountIn < sweepBatchSize) {
280                 return (_amountIn * bridgeFeeShare) / FEE_BASIS;
281             } else {
282                 return (sweepBatchSize * bridgeFeeShare) / FEE_BASIS;
283             }
292             if (address(oracle) != address(0)) {
293                 (uint256 oraclePrice, uint256 oracleTimestamp) = oracle.getMintRate();
294                 return
295                     oracleTimestamp > lastPriceTimestamp
296                         ? (oraclePrice, oracleTimestamp)
297                         : (lastPrice, lastPriceTimestamp);
298             } else {
299                 return (lastPrice, lastPriceTimestamp);
300             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


309             if (_limit == _maxLimit) {
310                 return _limit;
311             } else if (_timestamp + _DURATION <= block.timestamp) {
312                 _limit = _maxLimit;
313             } else if (_timestamp + _DURATION > block.timestamp) {
314                 uint256 _timePassed = block.timestamp - _timestamp;
315                 uint256 _calculatedLimit = _limit + (_timePassed * _ratePerSecond);
316                 _limit = _calculatedLimit > _maxLimit ? _maxLimit : _calculatedLimit;
317             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


157             if (_asset != IS_NATIVE) {
158                 return IERC20(_asset).balanceOf(address(this)) - claimReserve[_asset];
159             } else {
160                 return address(this).balance - claimReserve[_asset];
161             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

## NC009 - Events may be emitted out of order due to reentrancy:

Ensure that events follow the best practice of check-effects-interaction, and are emitted before external calls


<details>
<summary>Click to show 8 findings</summary>

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


/// @audit safeTransfer() called before event
126             emit Withdraw(_to, _amount);


/// @audit safeTransferFrom() called before event
151             emit Deposit(_to, _amount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L151:151

```solidity
File: contracts/Deposits/DepositQueue.sol


/// @audit safeTransfer() called before event
264                     emit ProtocolFeesPaid(token, feeAmount, feeAddress);


/// @audit safeTransfer() called before event
275                 emit RewardsDeposited(IERC20(address(token)), balance - feeAmount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L275:275

```solidity
File: contracts/RestakeManager.sol


/// @audit safeTransferFrom() called before event
575             emit Deposit(msg.sender, _collateralToken, _amount, ezETHToMint, _referralId);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L575:575

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


/// @audit safeTransferFrom() called before event
198             emit ERC20BufferFilled(_asset, _amount);


/// @audit safeTransferFrom() called before event
255             emit WithdrawRequestCreated(
256                 withdrawRequestNonce,
257                 msg.sender,
258                 _assetOut,
259                 amountToRedeem,
260                 _amount,
261                 withdrawRequests[msg.sender].length - 1
262             );


/// @audit transfer() called before event
311             emit WithdrawRequestClaimed(_withdrawRequest);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L311:311

</details>

## NC010 - If-statement can be converted to a ternary:

The code can be made more compact while also increasing readability by converting the following if-statements to ternaries (e.g. foo += (x > y) ? a : b)


```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


279             if (_amountIn < sweepBatchSize) {
280                 return (_amountIn * bridgeFeeShare) / FEE_BASIS;
281             } else {
282                 return (sweepBatchSize * bridgeFeeShare) / FEE_BASIS;
283             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


157             if (_asset != IS_NATIVE) {
158                 return IERC20(_asset).balanceOf(address(this)) - claimReserve[_asset];
159             } else {
160                 return address(this).balance - claimReserve[_asset];
161             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

## NC011 - Import declarations should import specific identifiers, rather than the whole file:

Using import declarations of the form import {<identifier_name>} from 'some/file.sol' avoids polluting the symbol namespace making flattened files smaller, and speeds up compilation


<details>
<summary>Click to show 23 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
14      import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridgeStorage.sol


5       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridgeStorage.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
15      import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


5       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


14      import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


11      import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
12      import "contest/node_modules/@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


9       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


11      import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
12      import "contest/node_modules/@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


4       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
5       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
7       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracleStorage.sol


6       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/IERC20.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracleStorage.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProviderStorage.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProviderStorage.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


4       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol";
6       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
7       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
8       import "contest/node_modules/@openzeppelin/contracts-upgradeable/token/ERC20/IERC20Upgradeable.sol";
9       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/IERC20.sol";
10      import "contest/node_modules/@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
6       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


6       import "contest/node_modules/@openzeppelin/contracts/access/AccessControl.sol";
7       import "contest/node_modules/@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";
8       import "contest/node_modules/@openzeppelin/contracts/token/ERC1155/IERC1155Receiver.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


6       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
7       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/PausableUpgradeable.sol";
8       import "contest/node_modules/@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol";
9       import "contest/node_modules/@openzeppelin/contracts/token/ERC20/IERC20.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


4       import "contest/node_modules/@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
5       import "contest/node_modules/@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>


## NC012 - Use a struct to encapsulate multiple function parameters:

Using a struct to encapsulate multiple parameters in Solidity functions can significantly enhance code readability and maintainability. Instead of passing a long list of arguments, which can be error-prone and hard to manage, a struct allows grouping related data into a single, coherent entity. This approach simplifies function signatures and makes the code more organized. It also enhances code clarity, as developers can easily understand the relationship between the parameters. Moreover, it aids in future code modifications and expansions, as adding or modifying a parameter only requires changes in the struct definition, rather than in every function that uses these parameters.


<details>
<summary>Click to show 11 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


18          function bridgeERC20To(
19              address _localToken,
20              address _remoteToken,
21              address _to,
22              uint256 _amount,
23              uint32 _minGasLimit,
24              bytes calldata _extraData
56          function bridgeTo(
57              address _to,
58              address _erc20,
59              address _remoteToken,
60              uint256 _amount,
61              uint32 _minGasLimit,
62              bytes calldata _extraData
63          ) external {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


70          function initialize(
71              IERC20 _ezETH,
72              IERC20 _xezETH,
73              IRestakeManager _restakeManager,
74              IERC20 _wETH,
75              IXERC20Lockbox _xezETHLockbox,
76              IConnext _connext,
77              IRouterClient _linkRouterClient,
78              IRateProvider _rateProvider,
79              LinkTokenInterface _linkToken,
80              IRoleManager _roleManager
81          ) public initializer {
139         function xReceive(
140             bytes32 _transferId,
141             uint256 _amount,
142             address _asset,
143             address _originSender,
144             uint32 _origin,
145             bytes memory
146         ) external nonReentrant returns (bytes memory) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


69          function xReceive(
70              bytes32 _transferId,
71              uint256,
72              address,
73              address _originSender,
74              uint32 _origin,
75              bytes memory _callData
76          ) external onlySource(_originSender, _origin) whenNotPaused returns (bytes memory) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


75          function initialize(
76              uint256 _currentPrice,
77              IERC20 _xezETH,
78              IERC20 _depositToken,
79              IERC20 _collateralToken,
80              IConnext _connext,
81              bytes32 _swapKey,
82              address _receiver,
83              uint32 _bridgeDestinationDomain,
84              address _bridgeTargetAddress,
85              IRenzoOracleL2 _oracle
86          ) public initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


74          function deployXERC20(
75              string memory _name,
76              string memory _symbol,
77              uint256[] memory _minterLimits,
78              uint256[] memory _burnerLimits,
79              address[] memory _bridges,
80              address _proxyAdmin
81          ) external returns (address _xerc20) {
135         function _deployXERC20(
136             string memory _name,
137             string memory _symbol,
138             uint256[] memory _minterLimits,
139             uint256[] memory _burnerLimits,
140             address[] memory _bridges,
141             address _proxyAdmin
142         ) internal returns (address _xerc20) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


35          function initialize(
36              string memory _name,
37              string memory _symbol,
38              address _factory,
39              address _l1Token,
40              address _optimismBridge
41          ) public initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


37          function deployOptimismMintableXERC20(
38              string memory _name,
39              string memory _symbol,
40              uint256[] memory _minterLimits,
41              uint256[] memory _burnerLimits,
42              address[] memory _bridges,
43              address _proxyAdmin,
44              address _l1Token
45          ) public returns (address _xerc20) {
73          function _deployOptimismMintableXERC20(
74              string memory _name,
75              string memory _symbol,
76              uint256[] memory _minterLimits,
77              uint256[] memory _burnerLimits,
78              address[] memory _bridges,
79              address _proxyAdmin,
80              address _l1Token
81          ) internal returns (address _xerc20) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


74          function initialize(
75              IRoleManager _roleManager,
76              IStrategyManager _strategyManager,
77              IRestakeManager _restakeManager,
78              IDelegationManager _delegationManager,
79              IEigenPodManager _eigenPodManager
80          ) external initializer {
364         function verifyWithdrawalCredentials(
365             uint64 oracleTimestamp,
366             BeaconChainProofs.StateRootProof calldata stateRootProof,
367             uint40[] calldata validatorIndices,
368             bytes[] calldata withdrawalCredentialProofs,
369             bytes32[][] calldata validatorFields
370         ) external onlyNativeEthRestakeAdmin {
405         function verifyAndProcessWithdrawals(
406             uint64 oracleTimestamp,
407             BeaconChainProofs.StateRootProof calldata stateRootProof,
408             BeaconChainProofs.WithdrawalProof[] calldata withdrawalProofs,
409             bytes[] calldata validatorFieldsProofs,
410             bytes32[][] calldata validatorFields,
411             bytes32[][] calldata withdrawalFields
412         ) external onlyNativeEthRestakeAdmin {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


101         function initialize(
102             IRoleManager _roleManager,
103             IEzEthToken _ezETH,
104             IRenzoOracle _renzoOracle,
105             IStrategyManager _strategyManager,
106             IDelegationManager _delegationManager,
107             IDepositQueue _depositQueue
108         ) public initializer {
400         function chooseOperatorDelegatorForWithdraw(
401             uint256 tokenIndex,
402             uint256 ezETHValue,
403             uint256[][] memory operatorDelegatorTokenTVLs,
404             uint256[] memory operatorDelegatorTVLs,
405             uint256 totalTVL
406         ) public view returns (IOperatorDelegator) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/TimelockController.sol


201         function hashOperation(
202             address target,
203             uint256 value,
204             bytes calldata data,
205             bytes32 predecessor,
206             bytes32 salt
207         ) public pure virtual returns (bytes32) {
215         function hashOperationBatch(
216             address[] calldata targets,
217             uint256[] calldata values,
218             bytes[] calldata payloads,
219             bytes32 predecessor,
220             bytes32 salt
221         ) public pure virtual returns (bytes32) {
234         function schedule(
235             address target,
236             uint256 value,
237             bytes calldata data,
238             bytes32 predecessor,
239             bytes32 salt,
240             uint256 delay
241         ) public virtual onlyRole(PROPOSER_ROLE) {
259         function scheduleBatch(
260             address[] calldata targets,
261             uint256[] calldata values,
262             bytes[] calldata payloads,
263             bytes32 predecessor,
264             bytes32 salt,
265             uint256 delay
266         ) public virtual onlyRole(PROPOSER_ROLE) {
315         function execute(
316             address target,
317             uint256 value,
318             bytes calldata payload,
319             bytes32 predecessor,
320             bytes32 salt
321         ) public payable virtual onlyRoleOrOpenRole(EXECUTOR_ROLE) {
342         function executeBatch(
343             address[] calldata targets,
344             uint256[] calldata values,
345             bytes[] calldata payloads,
346             bytes32 predecessor,
347             bytes32 salt
348         ) public payable virtual onlyRoleOrOpenRole(EXECUTOR_ROLE) {
423         function onERC1155Received(
424             address,
425             address,
426             uint256,
427             uint256,
428             bytes memory
429         ) public virtual override returns (bytes4) {
436         function onERC1155BatchReceived(
437             address,
438             address,
439             uint256[] memory,
440             uint256[] memory,
441             bytes memory
442         ) public virtual override returns (bytes4) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


64          function initialize(
65              IRoleManager _roleManager,
66              IRestakeManager _restakeManager,
67              IEzEthToken _ezETH,
68              IRenzoOracle _renzoOracle,
69              uint256 _coolDownPeriod,
70              TokenWithdrawBuffer[] calldata _withdrawalBufferTarget
71          ) external initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0


</details>

## NC013 - Constant redefined elsewhere:

Consider defining in only one contract so that values cannot become out of sync when only one location is updated. A cheap way to store constants in a single location is to create an internal constant in a library. If the variable is a local cache of another contract's value, consider making the cache variable internal or private, which will require external users to query the contract with the source of truth, so that callers don't get out of sync.


```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


    uint8 public constant EXPECTED_DECIMALS = 18;

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L37:37

```solidity
File: contracts/Deposits/DepositQueue.sol


    address public constant IS_NATIVE = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L13:13

```solidity
File: contracts/Withdraw/WithdrawQueueStorage.sol


    address public constant IS_NATIVE = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueueStorage.sol#L10:10

## NC014 - Multiple `address`/ID mappings can be combined into a single `mapping` of an `address`/ID to a `struct`, for readability:

Well-organized data structures make code reviews easier, which may lead to fewer bugs. Consider combining related mappings into mappings to structs, so it's clear what data is related


```solidity
File: contracts/Delegation/OperatorDelegatorStorage.sol


56          mapping(address => uint256) public adminGasSpentInWei;
57      }
58      
59      abstract contract OperatorDelegatorStorageV4 is OperatorDelegatorStorageV3 {
60          /// @dev mapping of token shares in withdraw queue of EigenLayer
61          mapping(address => uint256) public queuedShares;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegatorStorage.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueueStorage.sol


44          mapping(address => uint256) public withdrawalBufferTarget;
45      
46          /// @dev mapping of claimReserve (already withdraw requested), indexed by token address
47          mapping(address => uint256) public claimReserve;
48      
49          /// @dev mapiing of withdraw requests array, indexed by user address
50          mapping(address => WithdrawRequest[]) public withdrawRequests;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueueStorage.sol#L0:0

## NC015 - Duplicated `require()`/`revert()` checks should be refactored to a modifier or function:

The compiler will inline the function, which will avoid JUMP instructions usually associated with functions.


```solidity
File: contracts/TimelockController.sol


267             require(targets.length == values.length, "TimelockController: length mismatch");
268             require(targets.length == payloads.length, "TimelockController: length mismatch");
377             require(isOperationReady(id), "TimelockController: operation is not ready");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

## NC016 - Interfaces should be indicated with an I prefix in the contract name:

  


```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


17      interface L1StandardBridge {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0


## NC017 - Variable names that consist of all capital letters should be reserved for constant/immutable variables:

If the variable needs to be different based on which class it comes from, a view/pure function should be used instead.


```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


31          address public FACTORY;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


18          IXERC20 public XERC20;
23          IERC20 public ERC20;
29          bool public IS_NATIVE;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

## NC018 - Variable names for `immutable`s should use CONSTANT_CASE:

For `immutable` variable names, each word should use all capital letters, with underscores separating each word (CONSTANT_CASE).


```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


31          address immutable blastStandardBridge;
32          address immutable registry;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

## NC019 - Lines are too long:

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. The solidity style guide recommends a maximumum line length of 120 characters, so the lines below should be split when they reach that length.


<details>
<summary>Click to show 28 findings</summary>

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


// Verify that the pricing of the oracle less than or equal 18 decimals - pricing calculations will be off otherwise

// Verify that the pricing of the oracle is less than or equal to 18 decimals - pricing calculations will be off otherwise

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L38:38

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


* @dev     This function will receive the price feed and timestamp from the L1 through CCIPReceiver middleware contract.

// We will accept any amount of tokens out here... The caller of this function should verify the amount meets minimums

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L371:371

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


// constructor(string memory _name, string memory _symbol, address _factory) ERC20Upgradeable(_name, _symbol) ERC20PermitUpgradeable(_name) {

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L43:43

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


* @param _proxyAdmin The address of the proxy admin - will have permission to upgrade the lockbox (should be a dedicated account or contract to manage upgrades)

* @param _isNative Whether or not the base token is the native (gas) token of the chain. Eg: MATIC for polygon chain

* @param _proxyAdmin The address of the proxy admin - will have permission to upgrade the lockbox (should be a dedicated account or contract to manage upgrades)

* @param _proxyAdmin The address of the proxy admin - will have permission to upgrade the lockbox (should be a dedicated account or contract to manage upgrades)

* @param _isNative Whether or not the base token is the native (gas) token of the chain. Eg: MATIC for polygon chain

* @param _proxyAdmin The address of the proxy admin - will have permission to upgrade the lockbox (should be a dedicated account or contract to manage upgrades)

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L181:181

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


* @param _proxyAdmin The address of the proxy admin - will have permission to upgrade the lockbox (should be a dedicated account or contract to manage upgrades)

* @param _proxyAdmin The address of the proxy admin - will have permission to upgrade the lockbox (should be a dedicated account or contract to manage upgrades)

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L68:68

```solidity
File: contracts/Delegation/OperatorDelegator.sol


/// @dev Sets the strategy for a given token - setting strategy to 0x0 removes the ability to deposit and withdraw token

/// @dev Deposit tokens into the EigenLayer.  This call assumes any balance of tokens in this contract will be delegated

* @param   middlewareTimesIndex  is the index in the operator that the staker who triggered the withdrawal was delegated to's middleware times array

* @dev     Before the eigenpod is verified, we can sweep out any accumulated ETH from the Consensus layer validator rewards

*         We also want to track the amount in the delayed withdrawal router so we can track the TVL and reward amount accurately

* @dev Handle ETH sent to this contract - will get forwarded to the deposit queue for restaking as a protocol reward

* @dev If msg.sender is eigenPod then forward ETH to deposit queue without taking cut (i.e. full withdrawal from beacon chain)

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L499:499

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


* @dev     The contract hard codes the decimals to 18 decimals and returns the conversion rate of 1 mETH to ETH underlying the token in the wBETH protocol

* @notice  This contract is a shim that implements the Chainlink AggregatorV3Interface and returns pricing from the wBETH token contract

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L12:12

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


* @dev     The contract hard codes the decimals to 18 decimals and returns the conversion rate of 1 mETH to ETH underlying the token in the mETH protocol

* @notice  This contract is a shim that implements the Chainlink AggregatorV3Interface and returns pricing from the mETH staking contract

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L12:12

```solidity
File: contracts/Oracle/RenzoOracle.sol


// @dev Given list of tokens and balances, return total value (assumes all lookups are denomintated in same underlying currency)

/// @dev Given amount of current protocol value, new value being added, and supply of ezETH, determine amount to mint

// Given the amount of ezETH to burn, the supply of ezETH, and the total value in the protocol, determine amount of value to return to user

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L151:151

```solidity
File: contracts/RestakeManager.sol


/// @dev This function calculates the TVLs for each operator delegator by individual token, total for each OD, and total for the protocol.

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L270:270

</details>

## NC020 - Inconsistent spacing in comments:

Some lines use // x and some use //x. The instances below point out the usages that don't follow the majority, within each file.


```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


154             //set sweep batch size to 32 ETH


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

## NC021 - Use @inheritdoc rather than using a non-standard annotation:

Instead of referring to other code with @dev See {some-code}, use @inheritdoc.


```solidity
File: contracts/TimelockController.sol


140          * @dev See {IERC165-supportsInterface}.
409          * @dev See {IERC721Receiver-onERC721Received}.
421          * @dev See {IERC1155Receiver-onERC1155Received}.
434          * @dev See {IERC1155Receiver-onERC1155BatchReceived}.


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

## NC022 - Non-external/public variable names should begin with an underscore:

According to the Solidity Style Guide, non-external/public variable names should begin with an underscore


```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


31          address immutable blastStandardBridge;
32          address immutable registry;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0


## NC023 - Not using the named return variables anywhere in the function is confusing:

Consider changing the return variable to be an unnamed one, since the variable is never assigned, nor is it returned by name


```solidity
File: contracts/Delegation/OperatorDelegator.sol


146         ) external nonReentrant onlyRestakeManager returns (uint256 shares) {
162         function _deposit(IERC20 _token, uint256 _tokenAmount) internal returns (uint256 shares) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


49              returns (
50                  uint80 roundId,
51                  int256 answer,
52                  uint256 startedAt,
53                  uint256 updatedAt,
54                  uint80 answeredInRound
55              )
72              returns (
73                  uint80 roundId,
74                  int256 answer,
75                  uint256 startedAt,
76                  uint256 updatedAt,
77                  uint80 answeredInRound
78              )


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


49              returns (
50                  uint80 roundId,
51                  int256 answer,
52                  uint256 startedAt,
53                  uint256 updatedAt,
54                  uint80 answeredInRound
55              )
72              returns (
73                  uint80 roundId,
74                  int256 answer,
75                  uint256 startedAt,
76                  uint256 updatedAt,
77                  uint80 answeredInRound
78              )


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0



## NC024 - Expressions for constant values such as a call to `keccak256()`, should use `immutable` rather than `constant`:

While it **doesn't save any gas** because the compiler knows that developers often make this mistake, it's still best to use the right tool for the task at hand. There is a difference between `constant` variables and `immutable` variables, and they should each be used in their appropriate contexts. `constants` should be used for literal values written into the code, and `immutable` variables should be used for expressions, or values calculated in, or passed into the constructor.


```solidity
File: contracts/Permissions/RoleManagerStorage.sol


11          bytes32 public constant RX_ETH_MINTER_BURNER = keccak256("RX_ETH_MINTER_BURNER");
14          bytes32 public constant OPERATOR_DELEGATOR_ADMIN = keccak256("OPERATOR_DELEGATOR_ADMIN");
17          bytes32 public constant ORACLE_ADMIN = keccak256("ORACLE_ADMIN");
20          bytes32 public constant RESTAKE_MANAGER_ADMIN = keccak256("RESTAKE_MANAGER_ADMIN");
23          bytes32 public constant TOKEN_ADMIN = keccak256("TOKEN_ADMIN");
26          bytes32 public constant NATIVE_ETH_RESTAKE_ADMIN = keccak256("NATIVE_ETH_RESTAKE_ADMIN");
29          bytes32 public constant ERC20_REWARD_ADMIN = keccak256("ERC20_REWARD_ADMIN");
32          bytes32 public constant DEPOSIT_WITHDRAW_PAUSER = keccak256("DEPOSIT_WITHDRAW_PAUSER");
39          bytes32 public constant BRIDGE_ADMIN = keccak256("BRIDGE_ADMIN");
42          bytes32 public constant PRICE_FEED_SENDER = keccak256("PRICE_FEED_SENDER");
47          bytes32 public constant WITHDRAW_QUEUE_ADMIN = keccak256("WITHDRAW_QUEUE_ADMIN");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManagerStorage.sol#L0:0

```solidity
File: contracts/TimelockController.sol


26          bytes32 public constant TIMELOCK_ADMIN_ROLE = keccak256("TIMELOCK_ADMIN_ROLE");
27          bytes32 public constant PROPOSER_ROLE = keccak256("PROPOSER_ROLE");
28          bytes32 public constant EXECUTOR_ROLE = keccak256("EXECUTOR_ROLE");
29          bytes32 public constant CANCELLER_ROLE = keccak256("CANCELLER_ROLE");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

## NC025 - Contracts should have full test coverage:

While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.


```solidity
File: Various Files


None

```

## NC026 - Large or complicated code bases should implement invariant tests:

Large code bases, or code with lots of inline-assembly, complicated math, or complicated interactions between multiple contracts, should implement invariant fuzzing tests. Invariant fuzzers such as Echidna require the test writer to come up with invariants which should not be violated under any circumstances, and the fuzzer tests various inputs and function calls to ensure that the invariants always hold. Even code with 100% code coverage can still have bugs due to the order of the operations a user performs, and invariant fuzzers, with properly and extensively-written invariants, can close this testing gap significantly.


```solidity
File: Various Files


None

```

## NC027 - Consider using `block.number` instead of `block.timestamp`:

`block.timestamp` is vulnerable to miner manipulation and creates a potential front-running vulnerability. Consider using `block.number` instead.


<details>
<summary>Click to show 9 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


216             bytes memory _callData = abi.encode(exchangeRate, block.timestamp);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


52              if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


120             lastPriceTimestamp = block.timestamp;
248             if (block.timestamp > _lastPriceTimestamp + 1 days) {
261             if (block.timestamp > _deadline) {
321             return _updatePrice(price, block.timestamp);
350             if (_timestamp > block.timestamp) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


207             bridges[_bridge].minterParams.timestamp = block.timestamp;
219             bridges[_bridge].burnerParams.timestamp = block.timestamp;
242             bridges[_bridge].minterParams.timestamp = block.timestamp;
264             bridges[_bridge].burnerParams.timestamp = block.timestamp;
311             } else if (_timestamp + _DURATION <= block.timestamp) {
313             } else if (_timestamp + _DURATION > block.timestamp) {
314                 uint256 _timePassed = block.timestamp - _timestamp;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


80              return (0, int256(wBETHToken.exchangeRate()), 0, block.timestamp, 0);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


80              return (0, int256(methStaking.mETHToETH(1 ether)), 0, block.timestamp, 0);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


76              if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();
93              if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/TimelockController.sol


170             return timestamp > _DONE_TIMESTAMP && timestamp <= block.timestamp;
286             _timestamps[id] = block.timestamp + delay;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


248                     block.timestamp
287             if (block.timestamp - _withdrawRequest.createdAt < coolDownPeriod) revert EarlyClaim();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## NC028 - Consider bounding input array length:

The functions below take in an unbounded array, and make function calls for entries in the array. While the function will revert if it eventually runs out of gas, it may be a nicer user experience to `require()` that the length of the array is below some reasonable maximum, so that the user doesn't have to use up a full transaction's gas only to see that the transaction reverts.


<details>
<summary>Click to show 8 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


218             for (uint256 i = 0; i < _destinationParam.length; ) {
219                 Client.EVM2AnyMessage memory evm2AnyMessage = Client.EVM2AnyMessage({
220                     receiver: abi.encode(_destinationParam[i]._renzoReceiver), // ABI-encoded xRenzoDepsot contract address
221                     data: _callData, // ABI-encoded ezETH exchange rate with Timestamp
222                     tokenAmounts: new Client.EVMTokenAmount[](0), // Empty array indicating no tokens are being sent
223                     extraArgs: Client._argsToBytes(
224                         // Additional arguments, setting gas limit
225                         Client.EVMExtraArgsV1({ gasLimit: 200_000 })
226                     ),
227                     // Set the feeToken  address, indicating LINK will be used for fees
228                     feeToken: address(linkToken)
229                 });
230     
231                 // Get the fee required to send the message
232                 uint256 fees = linkRouterClient.getFee(
233                     _destinationParam[i].destinationChainSelector,
234                     evm2AnyMessage
235                 );
236     
237                 if (fees > linkToken.balanceOf(address(this)))
238                     revert NotEnoughBalance(linkToken.balanceOf(address(this)), fees);
239     
240                 // approve the Router to transfer LINK tokens on contract's behalf. It will spend the fees in LINK
241                 linkToken.approve(address(linkRouterClient), fees);
242     
243                 // Send the message through the router and store the returned message ID
244                 bytes32 messageId = linkRouterClient.ccipSend(
245                     _destinationParam[i].destinationChainSelector,
246                     evm2AnyMessage
247                 );
248     
249                 // Emit an event with message details
250                 emit MessageSent(
251                     messageId,
252                     _destinationParam[i].destinationChainSelector,
253                     _destinationParam[i]._renzoReceiver,
254                     exchangeRate,
255                     address(linkToken),
256                     fees
257                 );
258                 unchecked {
259                     ++i;
260                 }
261             }
264             for (uint256 i = 0; i < _connextDestinationParam.length; ) {
265                 connext.xcall{ value: _connextDestinationParam[i].relayerFee }(
266                     _connextDestinationParam[i].destinationDomainId,
267                     _connextDestinationParam[i]._renzoReceiver,
268                     address(0),
269                     msg.sender,
270                     0,
271                     0,
272                     _callData
273                 );
274     
275                 emit ConnextMessageSent(
276                     _connextDestinationParam[i].destinationDomainId,
277                     _connextDestinationParam[i]._renzoReceiver,
278                     exchangeRate,
279                     _connextDestinationParam[i].relayerFee
280                 );
281     
282                 unchecked {
283                     ++i;
284                 }
285             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


167             for (uint256 _i; _i < _bridgesLength; ++_i) {
168                 XERC20(_xerc20).setLimits(_bridges[_i], _minterLimits[_i], _burnerLimits[_i]);
169             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


109             for (uint256 _i; _i < _bridgesLength; ++_i) {
110                 OptimismMintableXERC20(_xerc20).setLimits(
111                     _bridges[_i],
112                     _minterLimits[_i],
113                     _burnerLimits[_i]
114                 );
115             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


209             for (uint256 i; i < tokens.length; ) {
210                 if (address(tokens[i]) == IS_NATIVE) {
211                     // set beaconChainEthStrategy for ETH
212                     queuedWithdrawalParams[0].strategies[i] = eigenPodManager.beaconChainETHStrategy();
213     
214                     // set shares for ETH
215                     queuedWithdrawalParams[0].shares[i] = tokenAmounts[i];
216                 } else {
217                     if (address(tokenStrategyMapping[tokens[i]]) == address(0))
218                         revert InvalidZeroInput();
219     
220                     // set the strategy of the token
221                     queuedWithdrawalParams[0].strategies[i] = tokenStrategyMapping[tokens[i]];
222     
223                     // set the equivalent shares for tokenAmount
224                     queuedWithdrawalParams[0].shares[i] = tokenStrategyMapping[tokens[i]]
225                         .underlyingToSharesView(tokenAmounts[i]);
226                 }
227     
228                 // set withdrawer as this contract address
229                 queuedWithdrawalParams[0].withdrawer = address(this);
230     
231                 // track shares of tokens withdraw for TVL
232                 queuedShares[address(tokens[i])] += queuedWithdrawalParams[0].shares[i];
233                 unchecked {
234                     ++i;
235                 }
236             }
277             for (uint256 i; i < tokens.length; ) {
278                 if (address(tokens[i]) == address(0)) revert InvalidZeroInput();
279     
280                 // deduct queued shares for tracking TVL
281                 queuedShares[address(tokens[i])] -= withdrawal.shares[i];
282     
283                 // check if token is not Native ETH
284                 if (address(tokens[i]) != IS_NATIVE) {
285                     // Check the withdraw buffer and fill if below buffer target
286                     uint256 bufferToFill = withdrawQueue.getBufferDeficit(address(tokens[i]));
287     
288                     // get balance of this contract
289                     uint256 balanceOfToken = tokens[i].balanceOf(address(this));
290                     if (bufferToFill > 0) {
291                         bufferToFill = (balanceOfToken <= bufferToFill) ? balanceOfToken : bufferToFill;
292     
293                         // update amount to send to the operator Delegator
294                         balanceOfToken -= bufferToFill;
295     
296                         // safe Approve for depositQueue
297                         tokens[i].safeApprove(address(restakeManager.depositQueue()), bufferToFill);
298     
299                         // fill Withdraw Buffer via depositQueue
300                         restakeManager.depositQueue().fillERC20withdrawBuffer(
301                             address(tokens[i]),
302                             bufferToFill
303                         );
304                     }
305     
306                     // Deposit remaining tokens back to eigenLayer
307                     if (balanceOfToken > 0) {
308                         _deposit(tokens[i], balanceOfToken);
309                     }
310                 }
311                 unchecked {
312                     ++i;
313                 }
314             }
381             for (uint256 i = 0; i < validatorFields.length; ) {
382                 uint64 validatorCurrentBalanceGwei = BeaconChainProofs.getEffectiveBalanceGwei(
383                     validatorFields[i]
384                 );
385                 stakedButNotVerifiedEth -= (validatorCurrentBalanceGwei * GWEI_TO_WEI);
386                 unchecked {
387                     ++i;
388                 }
389             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


227             for (uint256 i = 0; i < arrayLength; ) {
228                 // Send the ETH and the params through to the restake manager
229                 restakeManager.stakeEthInOperatorDelegator{ value: 32 ether }(
230                     operatorDelegators[i],
231                     pubkeys[i],
232                     signatures[i],
233                     depositDataRoots[i]
234                 );
235     
236                 emit ETHStakedFromQueue(
237                     operatorDelegators[i],
238                     pubkeys[i],
239                     32 ether,
240                     address(this).balance
241                 );
242     
243                 unchecked {
244                     ++i;
245                 }
246             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


111             for (uint256 i = 0; i < tokenLength; ) {
112                 totalValue += lookupTokenValue(_tokens[i], _balances[i]);
113                 unchecked {
114                     ++i;
115                 }
116             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/TimelockController.sol


107             for (uint256 i = 0; i < proposers.length; ++i) {
108                 _setupRole(PROPOSER_ROLE, proposers[i]);
109                 _setupRole(CANCELLER_ROLE, proposers[i]);
110             }
113             for (uint256 i = 0; i < executors.length; ++i) {
114                 _setupRole(EXECUTOR_ROLE, executors[i]);
115             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


88              for (uint256 i = 0; i < _withdrawalBufferTarget.length; ) {
89                  if (
90                      _withdrawalBufferTarget[i].asset == address(0) ||
91                      _withdrawalBufferTarget[i].bufferAmount == 0
92                  ) revert InvalidZeroInput();
93                  withdrawalBufferTarget[_withdrawalBufferTarget[i].asset] = _withdrawalBufferTarget[i]
94                      .bufferAmount;
95                  unchecked {
96                      ++i;
97                  }
98              }
110             for (uint256 i = 0; i < _newBufferTarget.length; ) {
111                 if (_newBufferTarget[i].asset == address(0) || _newBufferTarget[i].bufferAmount == 0)
112                     revert InvalidZeroInput();
113                 emit WithdrawBufferTargetUpdated(
114                     withdrawalBufferTarget[_newBufferTarget[i].asset],
115                     _newBufferTarget[i].bufferAmount
116                 );
117                 withdrawalBufferTarget[_newBufferTarget[i].asset] = _newBufferTarget[i].bufferAmount;
118                 unchecked {
119                     ++i;
120                 }
121             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## NC029 - Variables should be named in mixedCase style:

As the [Solidity Style Guide](https://docs.soliditylang.org/en/latest/style-guide.html#naming-styles) suggests: arguments, local variables and mutable state variables should be named in mixedCase style.


<details>
<summary>Click to show 17 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


178             uint256 ezETHAmount = ezETH.balanceOf(address(this)) - ezETHBalanceBeforeDeposit;
71              IERC20 _ezETH,
72              IERC20 _xezETH,
190             uint256 xezETHAmount = xezETH.balanceOf(address(this)) - xezETHBalanceBeforeDeposit;
25          event EzETHMinted(
26              bytes32 transferId,
27              uint256 amountDeposited,
28              uint32 origin,
29              address originSender,
30              uint256 ezETHMinted
31          );
74              IERC20 _wETH,
75              IXERC20Lockbox _xezETHLockbox,
184             uint256 xezETHBalanceBeforeDeposit = xezETH.balanceOf(address(this));
172             uint256 ezETHBalanceBeforeDeposit = ezETH.balanceOf(address(this));


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridgeStorage.sol


32          IXERC20Lockbox public xezETHLockbox;
23          IERC20 public ezETH;
20          IERC20 public xezETH;
29          IERC20 public wETH;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridgeStorage.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


134         function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {
96          function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


92          function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {
130         function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


253             uint256 xezETHAmount = (1e18 * amountOut) / _lastPrice;
375             uint256 amountNextWETH = connext.swapExact(
77              IERC20 _xezETH,


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


17          IERC20 public xezETH;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


31          address public FACTORY;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


23          IERC20 public ERC20;
18          IXERC20 public XERC20;
29          bool public IS_NATIVE;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


23          function initialize(IStakedTokenV2 _wBETHToken) public initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShimStorage.sol


8           IStakedTokenV2 public wBETHToken;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


153             uint256 _ezETHBeingBurned,
126             uint256 _existingEzETHSupply
139             uint256 newEzETHSupply = (_existingEzETHSupply * SCALE_FACTOR) /
154             uint256 _existingEzETHSupply,


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


19              IERC20Upgradeable _ezETHToken
31              (, , uint256 totalTVL) = restakeManager.calculateTVLs();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProviderStorage.sol


12          IERC20Upgradeable public ezETHToken;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProviderStorage.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


364             uint256 totalTVL
277             uint256 totalTVL = 0;
405             uint256 totalTVL
103             IEzEthToken _ezETH,
594             (, , uint256 totalTVL) = calculateTVLs();
291                 uint256 operatorTVL = 0;
501                 uint256[][] memory operatorDelegatorTokenTVLs,
502                 uint256[] memory operatorDelegatorTVLs,
503                 uint256 totalTVL
402             uint256 ezETHValue,
517                 uint256 currentTokenTVL = 0;
403             uint256[][] memory operatorDelegatorTokenTVLs,
605             uint256 ezETHToMint = renzoOracle.calculateMintAmount(
215         function setMaxDepositTVL(uint256 _maxDepositTVL) external onlyRestakeManagerAdmin {
275             uint256[][] memory operatorDelegatorTokenTVLs = new uint256[][](operatorDelegators.length);
50          event UserWithdrawStarted(
51              bytes32 withdrawalRoot,
52              address withdrawer,
53              IERC20 token,
54              uint256 amount,
55              uint256 ezETHToBurn
56          );
41          event Deposit(
42              address depositor,
43              IERC20 token,
44              uint256 amount,
45              uint256 ezETHMinted,
46              uint256 referralId
47          );
59          event UserWithdrawCompleted(
60              bytes32 withdrawalRoot,
61              address withdrawer,
62              IERC20 token,
63              uint256 amount,
64              uint256 ezETHBurned
65          );
404             uint256[] memory operatorDelegatorTVLs,
652             (, uint256[] memory operatorDelegatorTVLs, uint256 totalTVL) = calculateTVLs();
565             uint256 ezETHToMint = renzoOracle.calculateMintAmount(
276             uint256[] memory operatorDelegatorTVLs = new uint256[](operatorDelegators.length);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/RestakeManagerStorage.sol


30              uint256 ezETHToBurn;
58          uint256 public maxDepositTVL;
20          IEzEthToken public ezETH;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


67              IEzEthToken _ezETH,
217             (, , uint256 totalTVL) = restakeManager.calculateTVLs();
27          event WithdrawRequestCreated(
28              uint256 indexed withdrawRequestID,
29              address user,
30              address claimToken,
31              uint256 amountToRedeem,
32              uint256 ezETHAmountLocked,
33              uint256 withdrawRequestIndex
34          );


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueueStorage.sol


19              uint256 withdrawRequestID;
21              uint256 ezETHLocked;
29          IEzEthToken public ezETH;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueueStorage.sol#L0:0

</details>

## NC030 - Use allowlist/denylist rather than whitelist/blacklist:

Use alternative variants, e.g. allowlist/denylist instead of whitelist/blacklist.


<details>
<summary>Click to show 11 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


134          * @notice  WARNING: This function does NOT whitelist who can send funds from the L2 via Connext.  Users should NOT


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


415             // Verify the caller is whitelisted


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


49          /// @dev Allows only a whitelisted address to configure the contract
61          /// @dev Allows only a whitelisted address to configure the contract


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


43          /// @dev Allows only a whitelisted address to configure the contract
55          /// @dev Allows only a whitelisted address to trigger native ETH staking
61          /// @dev Allows only a whitelisted address to trigger ERC20 rewards sweeping


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Errors/Errors.sol


97      /// @dev Error when the origin address is not whitelisted


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Errors/Errors.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


28          /// @dev Allows only a whitelisted address to configure the contract


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


84          /// @dev Determines if the specified address has permission to set whitelisted origin in xRenzoBridge


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/Permissions/RoleManagerStorage.sol


38          /// @dev role for granting capability to update whitelisted origin in xRenzoBridge


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManagerStorage.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


70          /// @dev Allows only a whitelisted address to configure the contract
76          /// @dev Allows only a whitelisted address to set pause state


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


17          /// @dev Allows only a whitelisted address to trigger native ETH staking
23          /// @dev Allows only a whitelisted address to configure the contract


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


14          /// @dev Allows only a whitelisted address to mint or burn ezETH tokens
20          /// @dev Allows only a whitelisted address to pause or unpause the token


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## NC031 - Contracts containing only utility functions should be made into libraries:

  


<details>
<summary>Click to show 14 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridgeStorage.sol


18      abstract contract xRenzoBridgeStorageV1 is IxRenzoBridge {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridgeStorage.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol


7       abstract contract RenzoOracleL2StorageV1 is IRenzoOracleL2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


9       abstract contract xRenzoDepositStorageV1 is IxRenzoDeposit {
47      abstract contract xRenzoDepositStorageV2 is xRenzoDepositStorageV1 {
52      abstract contract xRenzoDepositStorageV3 is xRenzoDepositStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegatorStorage.sol


16      abstract contract OperatorDelegatorStorageV1 is IOperatorDelegator {
46      abstract contract OperatorDelegatorStorageV2 is OperatorDelegatorStorageV1 {
51      abstract contract OperatorDelegatorStorageV3 is OperatorDelegatorStorageV2 {
59      abstract contract OperatorDelegatorStorageV4 is OperatorDelegatorStorageV3 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegatorStorage.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueueStorage.sol


9       abstract contract DepositQueueStorageV1 is IDepositQueue {
26      abstract contract DepositQueueStorageV2 is DepositQueueStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueueStorage.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShimStorage.sol


7       abstract contract WBETHShimStorageV1 is AggregatorV3Interface {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShimStorage.sol


7       abstract contract METHShimStorageV1 is AggregatorV3Interface {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracleStorage.sol


8       abstract contract RenzoOracleStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracleStorage.sol#L0:0

```solidity
File: contracts/Permissions/RoleManagerStorage.sol


9       contract RoleManagerStorageV1 {
37      contract RoleManagerStorageV2 is RoleManagerStorageV1 {
45      contract RoleManagerStorageV3 is RoleManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManagerStorage.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProviderStorage.sol


7       abstract contract BalancerRateProviderStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProviderStorage.sol#L0:0

```solidity
File: contracts/RestakeManagerStorage.sol


15      abstract contract RestakeManagerStorageV1 is IRestakeManager {
64      abstract contract RestakeManagerStorageV2 is RestakeManagerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandlerStorage.sol


6       abstract contract RewardHandlerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandlerStorage.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueueStorage.sol


9       abstract contract WithdrawQueueStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueueStorage.sol#L0:0

```solidity
File: contracts/token/EzEthTokenStorage.sol


10      contract EzEthTokenStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthTokenStorage.sol#L0:0

</details>

## NC032 - Function names should use lowerCamelCase:

According to the Solidity [style guide](https://docs.soliditylang.org/en/latest/style-guide.html#function-names) function names should be in `mixedCase` (lowerCamelCase).


<details>
<summary>Click to show 11 findings</summary>

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


96          function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {
107         function updateCCIPEthChainSelector(uint64 _newChainSelector) external onlyOwner {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


92          function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {
103         function updateCCIPEthChainSelector(uint32 _newChainDomain) external onlyOwner {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


168         function depositETH(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


338         function getStakedETHBalance() external view returns (uint256) {
432         function withdrawNonBeaconChainETHBalanceWei(
459         function startDelayedWithdrawUnstakedETH() external onlyNativeEthRestakeAdmin {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


123         function depositETHFromProtocol() external payable onlyRestakeManager {
152         function forwardFullWithdrawalETH() external payable nonReentrant {
294         function _checkAndFillETHWithdrawBuffer(uint256 _amount) internal {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/IRestakeManager.sol


17          function calculateTVLs() external view returns (uint256[][] memory, uint256[] memory, uint256);
19          function depositETH() external payable;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/IRestakeManager.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


69          function _getWBETHData()


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


69          function _getMETHData()


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


38          function isEzETHMinterBurner(address potentialAddress) external view returns (bool) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


215         function setMaxDepositTVL(uint256 _maxDepositTVL) external onlyRestakeManagerAdmin {
274         function calculateTVLs() public view returns (uint256[][] memory, uint256[] memory, uint256) {
582         function depositETH() external payable {
592         function depositETH(uint256 _referralId) public payable nonReentrant notPaused {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


62          function _forwardETH() internal {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

</details>

## NC033 - Consider adding a deny-list:

Doing so will significantly increase centralization, but will help to prevent hackers from using stolen tokens.


<details>
<summary>Click to show 23 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


30      contract LockboxAdapterBlast {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


16      contract xRenzoBridge is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


11      contract RenzoOracleL2 is Initializable, OwnableUpgradeable, RenzoOracleL2StorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


14      contract Receiver is CCIPReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


10      contract ConnextReceiver is IXReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


27      contract xRenzoDeposit is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


16      contract XERC20 is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


14      contract XERC20Factory is Initializable, IXERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


11      contract XERC20Lockbox is Initializable, IXERC20Lockbox {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


11      contract OptimismMintableXERC20 is ERC165Upgradeable, XERC20, IOptimismMintableERC20 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


14      contract OptimismMintableXERC20Factory is Initializable, XERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


17      contract OperatorDelegator is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


10      contract DepositQueue is Initializable, ReentrancyGuardUpgradeable, DepositQueueStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


15      contract WBETHShim is Initializable, WBETHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


15      contract METHShim is Initializable, METHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


13      contract RenzoOracle is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


14      contract RoleManager is IRoleManager, AccessControlUpgradeable, RoleManagerStorageV3 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


9       contract BalancerRateProvider is Initializable, IRateProvider, BalancerRateProviderStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


26      contract RestakeManager is Initializable, ReentrancyGuardUpgradeable, RestakeManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


16      contract RewardHandler is Initializable, ReentrancyGuardUpgradeable, RewardHandlerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


25      contract TimelockController is AccessControl, IERC721Receiver, IERC1155Receiver {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


11      contract WithdrawQueue is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


13      contract EzEthToken is Initializable, ERC20Upgradeable, IEzEthToken, EzEthTokenStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>



## NC034 - Top level declarations should be separated by two blank lines:

  


<details>
<summary>Click to show 6 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


16      


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


51      
46      


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegatorStorage.sol


50      
45      
58      


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegatorStorage.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueueStorage.sol


25      


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueueStorage.sol#L0:0

```solidity
File: contracts/Permissions/RoleManagerStorage.sol


44      


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManagerStorage.sol#L0:0

```solidity
File: contracts/RestakeManagerStorage.sol


63      


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L0:0

</details>

## NC035 - Custom error has no error details:

Consider adding parameters to the error to indicate which user or values caused the failure.


```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


36          error AmountLessThanZero();
37          error InvalidAddress();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


15          error OptimismMintableXERC20Factory_NoBridges();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Errors/Errors.sol


5       error InvalidZeroInput();
8       error AlreadyAdded();
11      error NotFound();
14      error MaxTVLReached();
17      error NotRestakeManagerAdmin();
20      error NotDepositQueue();
23      error ContractPaused();
26      error OverMaxBasisPoints();
32      error WithdrawAlreadyCompleted();
38      error NotOperatorDelegatorAdmin();
41      error NotOracleAdmin();
44      error NotRestakeManager();
47      error NotNativeEthRestakeAdmin();
50      error DelegateAddressAlreadySet();
53      error NotERC20RewardsAdmin();
56      error TransferFailed();
59      error NotEzETHMinterBurner();
62      error NotTokenAdmin();
65      error OracleNotFound();
68      error OraclePriceExpired();
71      error MismatchedArrayLengths();
74      error NotDepositWithdrawPauser();
77      error MaxTokenTVLReached();
80      error InvalidOraclePrice();
83      error NotImplemented();
86      error InvalidTokenAmount();
92      error InsufficientOutputAmount();
95      error InvalidTokenReceived();
98      error InvalidOrigin();
104     error InvalidZeroOutput();
113     error UnauthorizedBridgeSweeper();
116     error NotBridgeAdmin();
119     error NotPriceFeedSender();
122     error UnAuthorisedCall();
125     error PriceFeedNotAvailable();
134     error NotWithdrawQueueAdmin();
137     error NotEnoughWithdrawBuffer();
140     error EarlyClaim();
143     error UnsupportedWithdrawAsset();
146     error InvalidWithdrawIndex();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Errors/Errors.sol#L0:0

## NC036 - Contract uses both `require()`/`revert()` as well as custom errors:

Consider using just one method in a single file


```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


30      contract LockboxAdapterBlast {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


14      contract OptimismMintableXERC20Factory is Initializable, XERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

## NC037 - Mixed usage of `int`/`uint` with `int256`/`uint256`:

`int256`/`uint256` are the preferred type names (they're what are used for function signatures), so they should be used consistently.


```solidity
File: contracts/Delegation/OperatorDelegator.sol


37              uint nonce,
38              uint startBlock,


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

## NC038 - Consider moving `msg.sender` checks to a common authorization `modifier`:

  


```solidity
File: contracts/TimelockController.sol


403             require(msg.sender == address(this), "TimelockController: caller must be timelock");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

## NC039 - Unused `struct` definition:

Note that there may be cases where a struct superficially appears to be used, but this is only because there are multiple definitions of the struct in different files. In such cases, the struct definition should be moved into a separate file. The instances below are the unused definitions.


```solidity
File: contracts/Bridge/Connext/libraries/LibConnextStorage.sol


struct ExecuteArgs {
    TransferInfo params;
    address[] routers;
    bytes[] routerSignatures;
    address sequencer;
    bytes sequencerSignature;
}

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/libraries/LibConnextStorage.sol#L64:70

```solidity
File: contracts/Bridge/Connext/libraries/TokenId.sol


struct TokenId {
    uint32 domain;
    bytes32 id;
}

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/libraries/TokenId.sol#L9:12

## NC040 - Unused `error` definition:

Note that there may be cases where an error superficially appears to be used, but this is only because there are multiple definitions of the error in different files. In such cases, the error definition should be moved into a separate file. The instances below are the unused definitions.


```solidity
File: contracts/Errors/Errors.sol


error WithdrawAlreadyCompleted();

error NotOriginalWithdrawCaller(address expectedCaller);

error InvalidOrigin();

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Errors/Errors.sol#L98:98

## NC041 - Events are missing sender information:

When an action is triggered based on a user's action, not being able to filter based on who triggered the action makes event processing a lot more cumbersome. Including the msg.sender the events of these types of action will make events much more useful to end users. Include `msg.sender` in the event output.


<details>
<summary>Click to show 14 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


196             emit EzETHMinted(_transferId, _amount, _origin, _originSender, ezETHAmount);
250                 emit MessageSent(
251                     messageId,
252                     _destinationParam[i].destinationChainSelector,
253                     _destinationParam[i]._renzoReceiver,
254                     exchangeRate,
255                     address(linkToken),
256                     fees
257                 );
275                 emit ConnextMessageSent(
276                     _connextDestinationParam[i].destinationDomainId,
277                     _connextDestinationParam[i]._renzoReceiver,
278                     exchangeRate,
279                     _connextDestinationParam[i].relayerFee
280                 );


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


42              emit OracleAddressUpdated(address(_oracleAddress), address(oracle));


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


98              emit XRenzoBridgeL1Updated(_newXRenzoBridgeL1, xRenzoBridgeL1);
109             emit CCIPEthChainSelectorUpdated(_newChainSelector, ccipEthChainSelector);
136             emit XRenzoDepositUpdated(_newXRenzoDeposit, address(xRenzoDeposit));


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


80              emit MessageReceived(_transferId, _origin, _originSender, _price, _timestamp);
94              emit XRenzoBridgeL1Updated(_newXRenzoBridgeL1, xRenzoBridgeL1);
105             emit ConnextEthChainDomainUpdated(_newChainDomain, connextEthChainDomain);
132             emit XRenzoDepositUpdated(_newXRenzoDeposit, address(xRenzoDeposit));


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


469             emit BridgeSweeperAddressUpdated(_sweeper, _allowed);
502             emit OraclePriceFeedUpdated(address(_oracle), address(oracle));
512             emit ReceiverPriceFeedUpdated(_receiver, receiver);
523             emit BridgeFeeShareUpdated(bridgeFeeShare, _newShare);
534             emit SweepBatchSizeUpdated(sweepBatchSize, _newBatchSize);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


142             emit BridgeLimitsSet(_mintingLimit, _burningLimit, _bridge);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


91              emit XERC20Deployed(_xerc20);
120             emit LockboxDeployed(_lockbox);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


113             emit TokenStrategyUpdated(_token, _strategy);
129             emit DelegationAddressUpdated(_delegateAddress);
136             emit BaseGasAmountSpentUpdated(baseGasAmountSpent, _baseGasAmountSpent);
241             emit WithdrawStarted(
242                 withdrawalRoot,
243                 address(this),
244                 delegateAddress,
245                 address(this),
246                 nonce,
247                 block.number,
248                 queuedWithdrawalParams[0].strategies,
249                 queuedWithdrawalParams[0].shares
250             );
317             emit WithdrawCompleted(
318                 delegationManager.calculateWithdrawalRoot(withdrawal),
319                 withdrawal.strategies,
320                 withdrawal.shares
321             );
523                 emit RewardsForwarded(destination, remainingAmount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


89              emit WithdrawQueueUpdated(address(withdrawQueue), address(_withdrawQueue));
108             emit FeeConfigUpdated(_feeAddress, _feeBasisPoints);
117             emit RestakeManagerUpdated(_restakeManager);
171                 emit ProtocolFeesPaid(IERC20(address(0x0)), feeAmount, feeAddress);
182             emit RewardsDeposited(IERC20(address(0x0)), remainingRewards);
202             emit ETHStakedFromQueue(operatorDelegator, pubkey, 32 ether, address(this).balance);
236                 emit ETHStakedFromQueue(
237                     operatorDelegators[i],
238                     pubkeys[i],
239                     32 ether,
240                     address(this).balance
241                 );
264                     emit ProtocolFeesPaid(token, feeAmount, feeAddress);
275                 emit RewardsDeposited(IERC20(address(token)), balance - feeAmount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


65              emit OracleAddressUpdated(_token, _oracleAddress);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


151             emit OperatorDelegatorAdded(_newOperatorDelegator);
156             emit OperatorDelegatorAllocationUpdated(_newOperatorDelegator, _allocationBasisPoints);
169                     emit OperatorDelegatorAllocationUpdated(_operatorDelegatorToRemove, 0);
174                     emit OperatorDelegatorRemoved(_operatorDelegatorToRemove);
211             emit OperatorDelegatorAllocationUpdated(_operatorDelegator, _allocationBasisPoints);
240             emit CollateralTokenAdded(_newCollateralToken);
253                     emit CollateralTokenRemoved(_collateralTokenToRemove);
716             emit CollateralTokenTvlUpdated(_token, _limit);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


79              emit RewardDestinationUpdated(_rewardDestination);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


404             emit MinDelayChange(_minDelay, newDelay);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


113                 emit WithdrawBufferTargetUpdated(
114                     withdrawalBufferTarget[_newBufferTarget[i].asset],
115                     _newBufferTarget[i].bufferAmount
116                 );
131             emit CoolDownPeriodUpdated(coolDownPeriod, _newCoolDownPeriod);
198             emit ERC20BufferFilled(_asset, _amount);
311             emit WithdrawRequestClaimed(_withdrawRequest);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>


## NC042 - Zero as a function argument should have a descriptive meaning:

Consider using descriptive constants or an enum instead of passing zero directly on function calls, as that might be error-prone, to fully describe the caller's intention.


<details>
<summary>Click to show 11 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


199             bytes memory returnData = new bytes(0);
222                     tokenAmounts: new Client.EVMTokenAmount[](0), // Empty array indicating no tokens are being sent
225                         Client.EVMExtraArgsV1({ gasLimit: 200_000 })
265                 connext.xcall{ value: _connextDestinationParam[i].relayerFee }(
266                     _connextDestinationParam[i].destinationDomainId,
267                     _connextDestinationParam[i]._renzoReceiver,
268                     address(0),
269                     msg.sender,
270                     0,
271                     0,
272                     _callData
273                 );


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


30              if (_oracle.decimals() > 18) revert InvalidTokenDecimals(18, _oracle.decimals());
40                  revert InvalidTokenDecimals(18, _oracleAddress.decimals());


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


434             connext.xcall{ value: msg.value }(
435                 bridgeDestinationDomain,
436                 bridgeTargetAddress,
437                 address(collateralToken),
438                 msg.sender,
439                 balance,
440                 0, // Asset is already nextWETH, so no slippage will be incurred
441                 bridgeCallData
442             );


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


163             _xerc20 = CREATE3.deploy(_salt, _bytecode, 0);
206             _lockbox = payable(CREATE3.deploy(_salt, _bytecode, 0));


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


105             _xerc20 = CREATE3.deploy(_salt, _bytecode, 0);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


201                 memory queuedWithdrawalParams = new IDelegationManager.QueuedWithdrawalParams[](1);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


202             emit ETHStakedFromQueue(operatorDelegator, pubkey, 32 ether, address(this).balance);
236                 emit ETHStakedFromQueue(
237                     operatorDelegators[i],
238                     pubkeys[i],
239                     32 ether,
240                     address(this).balance
241                 );


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


80              return (0, int256(methStaking.mETHToETH(1 ether)), 0, block.timestamp, 0);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


62                  revert InvalidTokenDecimals(18, _oracleAddress.decimals());


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


169                     emit OperatorDelegatorAllocationUpdated(_operatorDelegatorToRemove, 0);
232                 revert InvalidTokenDecimals(
233                     18,
234                     IERC20Metadata(address(_newCollateralToken)).decimals()
235                 );
474             deposit(_collateralToken, _amount, 0);
583             depositETH(0);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/TimelockController.sol


30          uint256 internal constant _DONE_TIMESTAMP = uint256(1);
30          uint256 internal constant _DONE_TIMESTAMP = uint256(1);
118             emit MinDelayChange(0, minDelay);
244             emit CallScheduled(id, 0, target, value, data, predecessor, delay);
245             if (salt != bytes32(0)) {
275             if (salt != bytes32(0)) {
326             emit CallExecuted(id, 0, target, value, payload);
379                 predecessor == bytes32(0) || isOperationDone(predecessor),


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

</details>

## NC042 - Function names should differ to make the code more readable:

In Solidity, while function overriding allows for functions with the same name to coexist, it is advisable to avoid this practice to enhance code readability and maintainability. Having multiple functions with the same name, even with different parameters or in inherited contracts, can cause confusion and increase the likelihood of errors during development, testing, and debugging. Using distinct and descriptive function names not only clarifies the purpose and behavior of each function, but also helps prevent unintended function calls or incorrect overriding. By adopting a clear and consistent naming convention, developers can create more comprehensible and maintainable smart contracts.


<details>
<summary>Click to show 90 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


    function initialize(
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
    ) public initializer {
        // Verify non-zero addresses on inputs
        if (
            address(_ezETH) == address(0) ||
            address(_xezETH) == address(0) ||
            address(_restakeManager) == address(0) ||
            address(_wETH) == address(0) ||
            address(_xezETHLockbox) == address(0) ||
            address(_connext) == address(0) ||
            address(_linkRouterClient) == address(0) ||
            address(_rateProvider) == address(0) ||
            address(_linkToken) == address(0) ||
            address(_roleManager) == address(0)
        ) {
            revert InvalidZeroInput();
        }

        // Verify all tokens have 18 decimals
        uint8 decimals = IERC20MetadataUpgradeable(address(_ezETH)).decimals();
        if (decimals != EXPECTED_DECIMALS) {
            revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
        }
        decimals = IERC20MetadataUpgradeable(address(_xezETH)).decimals();
        if (decimals != EXPECTED_DECIMALS) {
            revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
        }
        decimals = IERC20MetadataUpgradeable(address(_wETH)).decimals();
        if (decimals != EXPECTED_DECIMALS) {
            revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
        }
        decimals = IERC20MetadataUpgradeable(address(_linkToken)).decimals();
        if (decimals != EXPECTED_DECIMALS) {
            revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
        }

        // Save off inputs
        ezETH = _ezETH;
        xezETH = _xezETH;
        restakeManager = _restakeManager;
        wETH = _wETH;
        xezETHLockbox = _xezETHLockbox;
        connext = _connext;
        linkRouterClient = _linkRouterClient;
        rateProvider = _rateProvider;
        linkToken = _linkToken;
        roleManager = _roleManager;
    }

    function xReceive(
        bytes32 _transferId,
        uint256 _amount,
        address _asset,
        address _originSender,
        uint32 _origin,
        bytes memory
    ) external nonReentrant returns (bytes memory) {
        // Only allow incoming messages from the Connext contract
        if (msg.sender != address(connext)) {
            revert InvalidSender(address(connext), msg.sender);
        }

        // Check that the token received is wETH
        if (_asset != address(wETH)) {
            revert InvalidTokenReceived();
        }

        // Check that the amount sent is greater than 0
        if (_amount == 0) {
            revert InvalidZeroInput();
        }

        // Get the balance of ETH before the withdraw
        uint256 ethBalanceBeforeWithdraw = address(this).balance;

        // Unwrap the WETH
        IWeth(address(wETH)).withdraw(_amount);

        // Get the amount of ETH
        uint256 ethAmount = address(this).balance - ethBalanceBeforeWithdraw;

        // Get the amonut of ezETH before the deposit
        uint256 ezETHBalanceBeforeDeposit = ezETH.balanceOf(address(this));

        // Deposit it into Renzo RestakeManager
        restakeManager.depositETH{ value: ethAmount }();

        // Get the amount of ezETH that was minted
        uint256 ezETHAmount = ezETH.balanceOf(address(this)) - ezETHBalanceBeforeDeposit;

        // Approve the lockbox to spend the ezETH
        ezETH.safeApprove(address(xezETHLockbox), ezETHAmount);

        // Get the xezETH balance before the deposit
        uint256 xezETHBalanceBeforeDeposit = xezETH.balanceOf(address(this));

        // Send to the lockbox to be wrapped into xezETH
        xezETHLockbox.deposit(ezETHAmount);

        // Get the amount of xezETH that was minted
        uint256 xezETHAmount = xezETH.balanceOf(address(this)) - xezETHBalanceBeforeDeposit;

        // Burn it - it was already minted on the L2
        IXERC20(address(xezETH)).burn(address(this), xezETHAmount);

        // Emit the event
        emit EzETHMinted(_transferId, _amount, _origin, _originSender, ezETHAmount);

        // Return 0 for success
        bytes memory returnData = new bytes(0);
        return returnData;
    }

    function recoverNative(uint256 _amount, address _to) external onlyBridgeAdmin {
        payable(_to).transfer(_amount);
    }

    function recoverERC20(address _token, uint256 _amount, address _to) external onlyBridgeAdmin {
        IERC20(_token).safeTransfer(_to, _amount);
    }

    receive() external payable {}

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L313:313

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


    function initialize(AggregatorV3Interface _oracle) public initializer {
        // Initialize inherited classes
        __Ownable_init();

        if (address(_oracle) == address(0)) revert InvalidZeroInput();

        // Verify that the pricing of the oracle less than or equal 18 decimals - pricing calculations will be off otherwise
        if (_oracle.decimals() > 18) revert InvalidTokenDecimals(18, _oracle.decimals());

        oracle = _oracle;
    }

    function setOracleAddress(AggregatorV3Interface _oracleAddress) external onlyOwner {
        if (address(_oracleAddress) == address(0)) revert InvalidZeroInput();
        // Verify that the pricing of the oracle is less than or equal to 18 decimals - pricing calculations will be off otherwise
        if (_oracleAddress.decimals() > 18)
            revert InvalidTokenDecimals(18, _oracleAddress.decimals());

        emit OracleAddressUpdated(address(_oracleAddress), address(oracle));
        oracle = _oracleAddress;
    }

    function getMintRate() public view returns (uint256, uint256) {
        (, int256 price, , uint256 timestamp, ) = oracle.latestRoundData();
        if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();
        // scale the price to have 18 decimals
        uint256 _scaledPrice = (uint256(price)) * 10 ** (18 - oracle.decimals());
        if (_scaledPrice < 1 ether) revert InvalidOraclePrice();
        return (_scaledPrice, timestamp);
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L50:57

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


    function initialize(
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
        // Initialize inherited classes
        __Ownable_init();

        // Verify valid non zero values
        if (
            _currentPrice == 0 ||
            address(_xezETH) == address(0) ||
            address(_depositToken) == address(0) ||
            address(_collateralToken) == address(0) ||
            address(_connext) == address(0) ||
            _swapKey == 0 ||
            _bridgeDestinationDomain == 0 ||
            _bridgeTargetAddress == address(0)
        ) {
            revert InvalidZeroInput();
        }

        // Verify all tokens have 18 decimals
        uint8 decimals = IERC20MetadataUpgradeable(address(_depositToken)).decimals();
        if (decimals != EXPECTED_DECIMALS) {
            revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
        }
        decimals = IERC20MetadataUpgradeable(address(_collateralToken)).decimals();
        if (decimals != EXPECTED_DECIMALS) {
            revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
        }
        decimals = IERC20MetadataUpgradeable(address(_xezETH)).decimals();
        if (decimals != EXPECTED_DECIMALS) {
            revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
        }

        // Initialize the price and timestamp
        lastPrice = _currentPrice;
        lastPriceTimestamp = block.timestamp;

        // Set xezETH address
        xezETH = _xezETH;

        // Set the depoist token
        depositToken = _depositToken;

        // Set the collateral token
        collateralToken = _collateralToken;

        // Set the connext contract
        connext = _connext;

        // Set the swap key
        swapKey = _swapKey;

        // Set receiver contract address
        receiver = _receiver;
        // Connext router fee is 5 basis points
        bridgeRouterFeeBps = 5;

        // Set the bridge destination domain
        bridgeDestinationDomain = _bridgeDestinationDomain;

        // Set the bridge target address
        bridgeTargetAddress = _bridgeTargetAddress;

        // set oracle Price Feed struct
        oracle = _oracle;

        // set bridge Fee Share 0.05% where 100 basis point = 1%
        bridgeFeeShare = 5;

        //set sweep batch size to 32 ETH
        sweepBatchSize = 32 ether;
    }

    function recoverNative(uint256 _amount, address _to) external onlyOwner {
        payable(_to).transfer(_amount);
    }

    function recoverERC20(address _token, uint256 _amount, address _to) external onlyOwner {
        IERC20(_token).safeTransfer(_to, _amount);
    }

    receive() external payable {}

    function getMintRate() public view returns (uint256, uint256) {
        // revert if PriceFeedNotAvailable
        if (receiver == address(0) && address(oracle) == address(0)) revert PriceFeedNotAvailable();
        if (address(oracle) != address(0)) {
            (uint256 oraclePrice, uint256 oracleTimestamp) = oracle.getMintRate();
            return
                oracleTimestamp > lastPriceTimestamp
                    ? (oraclePrice, oracleTimestamp)
                    : (lastPrice, lastPriceTimestamp);
        } else {
            return (lastPrice, lastPriceTimestamp);
        }
    }

    function depositETH(
        uint256 _minOut,
        uint256 _deadline
    ) external payable nonReentrant returns (uint256) {
        if (msg.value == 0) {
            revert InvalidZeroInput();
        }

        // Get the deposit token balance before
        uint256 depositBalanceBefore = depositToken.balanceOf(address(this));

        // Wrap the deposit ETH to WETH
        IWeth(address(depositToken)).deposit{ value: msg.value }();

        // Get the amount of tokens that were wrapped
        uint256 wrappedAmount = depositToken.balanceOf(address(this)) - depositBalanceBefore;

        // Sanity check for 0
        if (wrappedAmount == 0) {
            revert InvalidZeroOutput();
        }

        return _deposit(wrappedAmount, _minOut, _deadline);
    }

    function deposit(
        uint256 _amountIn,
        uint256 _minOut,
        uint256 _deadline
    ) external nonReentrant returns (uint256) {
        if (_amountIn == 0) {
            revert InvalidZeroInput();
        }

        // Transfer deposit tokens from user to this contract
        depositToken.safeTransferFrom(msg.sender, address(this), _amountIn);

        return _deposit(_amountIn, _minOut, _deadline);
    }

    function _deposit(
        uint256 _amountIn,
        uint256 _minOut,
        uint256 _deadline
    ) internal returns (uint256) {
        // calculate bridgeFee for deposit amount
        uint256 bridgeFee = getBridgeFeeShare(_amountIn);
        // subtract from _amountIn and add to bridgeFeeCollected
        _amountIn -= bridgeFee;
        bridgeFeeCollected += bridgeFee;

        // Trade deposit tokens for nextWETH
        uint256 amountOut = _trade(_amountIn, _deadline);
        if (amountOut == 0) {
            revert InvalidZeroOutput();
        }

        // Fetch price and timestamp of ezETH from the configured price feed
        (uint256 _lastPrice, uint256 _lastPriceTimestamp) = getMintRate();

        // Verify the price is not stale
        if (block.timestamp > _lastPriceTimestamp + 1 days) {
            revert OraclePriceExpired();
        }

        // Calculate the amount of xezETH to mint - assumes 18 decimals for price and token
        uint256 xezETHAmount = (1e18 * amountOut) / _lastPrice;

        // Check that the user will get the minimum amount of xezETH
        if (xezETHAmount < _minOut) {
            revert InsufficientOutputAmount();
        }

        // Verify the deadline has not passed
        if (block.timestamp > _deadline) {
            revert InvalidTimestamp(_deadline);
        }

        // Mint xezETH to the user
        IXERC20(address(xezETH)).mint(msg.sender, xezETHAmount);

        // Emit the event and return amount minted
        emit Deposit(msg.sender, _amountIn, xezETHAmount);
        return xezETHAmount;
    }

    function getRate() external view override returns (uint256) {
        return lastPrice;
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L456:458

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


    function initialize(
        string memory _name,
        string memory _symbol,
        address _factory
    ) public initializer {
        __XERC20_init(_name, _symbol, _factory);
    }

    function mint(address _user, uint256 _amount) public virtual {
        _mintWithCaller(msg.sender, _user, _amount);
    }

    function burn(address _user, uint256 _amount) public virtual {
        if (msg.sender != _user) {
            _spendAllowance(_user, msg.sender, _amount);
        }

        _burnWithCaller(msg.sender, _user, _amount);
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L107:113

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


    function initialize(
        address _lockboxImplementation,
        address _xerc20Implementation
    ) public initializer {
        lockboxImplementation = _lockboxImplementation;
        xerc20Implementation = _xerc20Implementation;
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L54:60

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


    function initialize(address _xerc20, address _erc20, bool _isNative) public initializer {
        XERC20 = IXERC20(_xerc20);
        ERC20 = IERC20(_erc20);
        IS_NATIVE = _isNative;
    }

    receive() external payable {
        depositNative();
    }

    function deposit(uint256 _amount) external {
        if (IS_NATIVE) revert IXERC20Lockbox_Native();

        _deposit(msg.sender, _amount);
    }

    function _deposit(address _to, uint256 _amount) internal {
        if (!IS_NATIVE) {
            ERC20.safeTransferFrom(msg.sender, address(this), _amount);
        }

        XERC20.mint(_to, _amount);
        emit Deposit(_to, _amount);
    }

    function withdraw(uint256 _amount) external {
        _withdraw(msg.sender, _amount);
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L103:105

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


    function initialize(
        string memory _name,
        string memory _symbol,
        address _factory,
        address _l1Token,
        address _optimismBridge
    ) public initializer {
        __ERC165_init();
        __XERC20_init(_name, _symbol, _factory);
        l1Token = _l1Token;
        optimismBridge = _optimismBridge;
    }

    function mint(address _to, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
        XERC20.mint(_to, _amount);
    }

    function burn(address _from, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
        XERC20.burn(_from, _amount);
    }

    function supportsInterface(
        bytes4 interfaceId
    ) public view override(ERC165Upgradeable) returns (bool) {
        return
            interfaceId == type(IOptimismMintableERC20).interfaceId ||
            super.supportsInterface(interfaceId);
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L48:54

```solidity
File: contracts/Delegation/OperatorDelegator.sol


    function initialize(
        IRoleManager _roleManager,
        IStrategyManager _strategyManager,
        IRestakeManager _restakeManager,
        IDelegationManager _delegationManager,
        IEigenPodManager _eigenPodManager
    ) external initializer {
        if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
        if (address(_strategyManager) == address(0x0)) revert InvalidZeroInput();
        if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
        if (address(_delegationManager) == address(0x0)) revert InvalidZeroInput();
        if (address(_eigenPodManager) == address(0x0)) revert InvalidZeroInput();

        __ReentrancyGuard_init();

        roleManager = _roleManager;
        strategyManager = _strategyManager;
        restakeManager = _restakeManager;
        delegationManager = _delegationManager;
        eigenPodManager = _eigenPodManager;

        // Deploy new EigenPod
        eigenPod = IEigenPod(eigenPodManager.createPod());
    }

    receive() external payable nonReentrant {
        // check if sender contract is EigenPod. forward full withdrawal eth received
        if (msg.sender == address(eigenPod)) {
            restakeManager.depositQueue().forwardFullWithdrawalETH{ value: msg.value }();
        } else {
            // considered as protocol reward
            uint256 gasRefunded = 0;
            uint256 remainingAmount = msg.value;
            if (adminGasSpentInWei[tx.origin] > 0) {
                gasRefunded = _refundGas();
                // update the remaining amount
                remainingAmount -= gasRefunded;
                // If no funds left, return
                if (remainingAmount == 0) {
                    return;
                }
            }
            // Forward remaining balance to the deposit queue
            address destination = address(restakeManager.depositQueue());
            (bool success, ) = destination.call{ value: remainingAmount }("");
            if (!success) revert TransferFailed();

            emit RewardsForwarded(destination, remainingAmount);
        }
    }

    function deposit(
        IERC20 token,
        uint256 tokenAmount
    ) external nonReentrant onlyRestakeManager returns (uint256 shares) {
        if (address(tokenStrategyMapping[token]) == address(0x0) || tokenAmount == 0)
            revert InvalidZeroInput();

        // Move the tokens into this contract
        token.safeTransferFrom(msg.sender, address(this), tokenAmount);

        return _deposit(token, tokenAmount);
    }

    function _deposit(IERC20 _token, uint256 _tokenAmount) internal returns (uint256 shares) {
        // Approve the strategy manager to spend the tokens
        _token.safeApprove(address(strategyManager), _tokenAmount);

        // Deposit the tokens via the strategy manager
        return
            strategyManager.depositIntoStrategy(tokenStrategyMapping[_token], _token, _tokenAmount);
    }

    function _refundGas() internal returns (uint256) {
        uint256 gasRefund = address(this).balance >= adminGasSpentInWei[tx.origin]
            ? adminGasSpentInWei[tx.origin]
            : address(this).balance;
        bool success = payable(tx.origin).send(gasRefund);
        if (!success) revert TransferFailed();

        // reset gas spent by admin
        adminGasSpentInWei[tx.origin] -= gasRefund;

        emit GasRefunded(tx.origin, gasRefund);
        return gasRefund;
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L481:493

```solidity
File: contracts/Deposits/DepositQueue.sol


    function initialize(IRoleManager _roleManager) public initializer {
        __ReentrancyGuard_init();

        if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();

        roleManager = _roleManager;
    }

    receive() external payable nonReentrant {
        uint256 feeAmount = 0;
        // Take protocol cut of rewards if enabled
        if (feeAddress != address(0x0) && feeBasisPoints > 0) {
            feeAmount = (msg.value * feeBasisPoints) / 10000;
            (bool success, ) = feeAddress.call{ value: feeAmount }("");
            if (!success) revert TransferFailed();

            emit ProtocolFeesPaid(IERC20(address(0x0)), feeAmount, feeAddress);
        }
        // update remaining rewards
        uint256 remainingRewards = msg.value - feeAmount;
        // Check and fill ETH withdraw buffer if required
        _checkAndFillETHWithdrawBuffer(remainingRewards);

        // Add to the total earned
        totalEarned[address(0x0)] = totalEarned[address(0x0)] + remainingRewards;

        // Emit the rewards event
        emit RewardsDeposited(IERC20(address(0x0)), remainingRewards);
    }

    function _refundGas(uint256 initialGas) internal {
        uint256 gasUsed = (initialGas - gasleft()) * tx.gasprice;
        uint256 gasRefund = address(this).balance >= gasUsed ? gasUsed : address(this).balance;
        (bool success, ) = payable(msg.sender).call{ value: gasRefund }("");
        if (!success) revert TransferFailed();
        emit GasRefunded(msg.sender, gasRefund);
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L283:289

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


    function initialize(IStakedTokenV2 _wBETHToken) public initializer {
        if (address(_wBETHToken) == address(0x0)) revert InvalidZeroInput();

        wBETHToken = _wBETHToken;
    }

    function decimals() external pure returns (uint8) {
        return 18;
    }

    function description() external pure returns (string memory) {
        return "wBETH Chainlink Shim";
    }

    function version() external pure returns (uint256) {
        return 1;
    }

    function getRoundData(uint80) external pure returns (uint80, int256, uint256, uint256, uint80) {
        revert NotImplemented();
    }

    function latestRoundData()
        external
        view
        returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        )
    {
        return _getWBETHData();
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L46:58

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


    function initialize(IMethStaking _methStaking) public initializer {
        if (address(_methStaking) == address(0x0)) revert InvalidZeroInput();

        methStaking = _methStaking;
    }

    function decimals() external pure returns (uint8) {
        return 18;
    }

    function description() external pure returns (string memory) {
        return "METH Chainlink Shim";
    }

    function version() external pure returns (uint256) {
        return 1;
    }

    function getRoundData(uint80) external pure returns (uint80, int256, uint256, uint256, uint80) {
        revert NotImplemented();
    }

    function latestRoundData()
        external
        view
        returns (
            uint80 roundId,
            int256 answer,
            uint256 startedAt,
            uint256 updatedAt,
            uint80 answeredInRound
        )
    {
        return _getMETHData();
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L46:58

```solidity
File: contracts/Oracle/RenzoOracle.sol


    function initialize(IRoleManager _roleManager) public initializer {
        if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();

        __ReentrancyGuard_init();

        roleManager = _roleManager;
    }

    function setOracleAddress(
        IERC20 _token,
        AggregatorV3Interface _oracleAddress
    ) external nonReentrant onlyOracleAdmin {
        if (address(_token) == address(0x0)) revert InvalidZeroInput();

        // Verify that the pricing of the oracle is 18 decimals - pricing calculations will be off otherwise
        if (_oracleAddress.decimals() != 18)
            revert InvalidTokenDecimals(18, _oracleAddress.decimals());

        tokenOracleLookup[_token] = _oracleAddress;
        emit OracleAddressUpdated(_token, _oracleAddress);
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L54:66

```solidity
File: contracts/Permissions/RoleManager.sol


    function initialize(address roleManagerAdmin) public initializer {
        if (address(roleManagerAdmin) == address(0x0)) revert InvalidZeroInput();

        __AccessControl_init();

        _grantRole(DEFAULT_ADMIN_ROLE, roleManagerAdmin);
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L22:28

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


    function initialize(
        IRestakeManager _restakeManager,
        IERC20Upgradeable _ezETHToken
    ) public initializer {
        if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
        if (address(_ezETHToken) == address(0x0)) revert InvalidZeroInput();

        restakeManager = _restakeManager;
        ezETHToken = _ezETHToken;
    }

    function getRate() external view returns (uint256) {
        // Get the total TVL priced in ETH from restakeManager
        (, , uint256 totalTVL) = restakeManager.calculateTVLs();

        // Get the total supply of the ezETH token
        uint256 totalSupply = ezETHToken.totalSupply();

        // Sanity check
        if (totalSupply == 0 || totalTVL == 0) revert InvalidZeroInput();

        // Return the rate
        return (10 ** 18 * totalTVL) / totalSupply;
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L29:41

```solidity
File: contracts/RestakeManager.sol


    function initialize(
        IRoleManager _roleManager,
        IEzEthToken _ezETH,
        IRenzoOracle _renzoOracle,
        IStrategyManager _strategyManager,
        IDelegationManager _delegationManager,
        IDepositQueue _depositQueue
    ) public initializer {
        __ReentrancyGuard_init();

        roleManager = _roleManager;
        ezETH = _ezETH;
        renzoOracle = _renzoOracle;
        strategyManager = _strategyManager;
        delegationManager = _delegationManager;
        depositQueue = _depositQueue;
        paused = false;
    }

    function depositETH() external payable {
        depositETH(0);
    }

    function depositETH(uint256 _referralId) public payable nonReentrant notPaused {
        // Get the total TVL
        (, , uint256 totalTVL) = calculateTVLs();

        // Enforce TVL limit if set
        if (maxDepositTVL != 0 && totalTVL + msg.value > maxDepositTVL) {
            revert MaxTVLReached();
        }

        // Deposit the remaining ETH into the DepositQueue
        depositQueue.depositETHFromProtocol{ value: msg.value }();

        // Calculate how much ezETH to mint
        uint256 ezETHToMint = renzoOracle.calculateMintAmount(
            totalTVL,
            msg.value,
            ezETH.totalSupply()
        );

        // Mint the ezETH
        ezETH.mint(msg.sender, ezETHToMint);

        // Emit the deposit event
        emit Deposit(msg.sender, IERC20(address(0x0)), msg.value, ezETHToMint, _referralId);
    }

    function deposit(IERC20 _collateralToken, uint256 _amount) external {
        deposit(_collateralToken, _amount, 0);
    }

    function deposit(
        IERC20 _collateralToken,
        uint256 _amount,
        uint256 _referralId
    ) public nonReentrant notPaused {
        // Verify collateral token is in the list - call will revert if not found
        uint256 tokenIndex = getCollateralTokenIndex(_collateralToken);

        // Get the TVLs for each operator delegator and the total TVL
        (
            uint256[][] memory operatorDelegatorTokenTVLs,
            uint256[] memory operatorDelegatorTVLs,
            uint256 totalTVL
        ) = calculateTVLs();

        // Get the value of the collateral token being deposited
        uint256 collateralTokenValue = renzoOracle.lookupTokenValue(_collateralToken, _amount);

        // Enforce TVL limit if set, 0 means the check is not enabled
        if (maxDepositTVL != 0 && totalTVL + collateralTokenValue > maxDepositTVL) {
            revert MaxTVLReached();
        }

        // Enforce individual token TVL limit if set, 0 means the check is not enabled
        if (collateralTokenTvlLimits[_collateralToken] != 0) {
            // Track the current token's TVL
            uint256 currentTokenTVL = 0;

            // For each OD, add up the token TVLs
            uint256 odLength = operatorDelegatorTokenTVLs.length;
            for (uint256 i = 0; i < odLength; ) {
                currentTokenTVL += operatorDelegatorTokenTVLs[i][tokenIndex];
                unchecked {
                    ++i;
                }
            }

            // Check if it is over the limit
            if (currentTokenTVL + collateralTokenValue > collateralTokenTvlLimits[_collateralToken])
                revert MaxTokenTVLReached();
        }

        // Determine which operator delegator to use
        IOperatorDelegator operatorDelegator = chooseOperatorDelegatorForDeposit(
            operatorDelegatorTVLs,
            totalTVL
        );

        // Transfer the collateral token to this address
        _collateralToken.safeTransferFrom(msg.sender, address(this), _amount);

        // Check the withdraw buffer and fill if below buffer target
        uint256 bufferToFill = depositQueue.withdrawQueue().getBufferDeficit(
            address(_collateralToken)
        );
        if (bufferToFill > 0) {
            bufferToFill = (_amount <= bufferToFill) ? _amount : bufferToFill;
            // update amount to send to the operator Delegator
            _amount -= bufferToFill;

            // safe Approve for depositQueue
            _collateralToken.safeApprove(address(depositQueue), bufferToFill);

            // fill Withdraw Buffer via depositQueue
            depositQueue.fillERC20withdrawBuffer(address(_collateralToken), bufferToFill);
        }

        // Approve the tokens to the operator delegator
        _collateralToken.safeApprove(address(operatorDelegator), _amount);

        // Call deposit on the operator delegator
        operatorDelegator.deposit(_collateralToken, _amount);

        // Calculate how much ezETH to mint
        uint256 ezETHToMint = renzoOracle.calculateMintAmount(
            totalTVL,
            collateralTokenValue,
            ezETH.totalSupply()
        );

        // Mint the ezETH
        ezETH.mint(msg.sender, ezETHToMint);

        // Emit the deposit event
        emit Deposit(msg.sender, _collateralToken, _amount, ezETHToMint, _referralId);
    }

    function stakeEthInOperatorDelegator(
        IOperatorDelegator operatorDelegator,
        bytes calldata pubkey,
        bytes calldata signature,
        bytes32 depositDataRoot
    ) external payable onlyDepositQueue {
        // Verify the OD is in the list
        bool found = false;
        uint256 odLength = operatorDelegators.length;
        for (uint256 i = 0; i < odLength; ) {
            if (operatorDelegators[i] == operatorDelegator) {
                found = true;
                break;
            }

            unchecked {
                ++i;
            }
        }
        if (!found) revert NotFound();

        // Call the OD to stake the ETH
        operatorDelegator.stakeEth{ value: msg.value }(pubkey, signature, depositDataRoot);
    }

    function depositTokenRewardsFromProtocol(
        IERC20 _token,
        uint256 _amount
    ) external onlyDepositQueue {
        // Get the TVLs for each operator delegator and the total TVL
        (, uint256[] memory operatorDelegatorTVLs, uint256 totalTVL) = calculateTVLs();

        // Determine which operator delegator to use
        IOperatorDelegator operatorDelegator = chooseOperatorDelegatorForDeposit(
            operatorDelegatorTVLs,
            totalTVL
        );

        // Transfer the tokens to this address
        _token.safeTransferFrom(msg.sender, address(this), _amount);

        // Approve the tokens to the operator delegator
        _token.safeApprove(address(operatorDelegator), _amount);

        // Deposit the tokens into EigenLayer
        operatorDelegator.deposit(_token, _amount);
    }

    function calculateTVLs() public view returns (uint256[][] memory, uint256[] memory, uint256) {
        uint256[][] memory operatorDelegatorTokenTVLs = new uint256[][](operatorDelegators.length);
        uint256[] memory operatorDelegatorTVLs = new uint256[](operatorDelegators.length);
        uint256 totalTVL = 0;

        // Iterate through the ODs
        uint256 odLength = operatorDelegators.length;

        // flag for withdrawal queue balance set
        bool withdrawQueueTokenBalanceRecorded = false;
        address withdrawQueue = address(depositQueue.withdrawQueue());

        // withdrawalQueue total value
        uint256 totalWithdrawalQueueValue = 0;

        for (uint256 i = 0; i < odLength; ) {
            // Track the TVL for this OD
            uint256 operatorTVL = 0;

            // Track the individual token TVLs for this OD - native ETH will be last item in the array
            uint256[] memory operatorValues = new uint256[](collateralTokens.length + 1);
            operatorDelegatorTokenTVLs[i] = operatorValues;

            // Iterate through the tokens and get the value of each
            uint256 tokenLength = collateralTokens.length;
            for (uint256 j = 0; j < tokenLength; ) {
                // Get the value of this token

                uint256 operatorBalance = operatorDelegators[i].getTokenBalanceFromStrategy(
                    collateralTokens[j]
                );

                // Set the value in the array for this OD
                operatorValues[j] = renzoOracle.lookupTokenValue(
                    collateralTokens[j],
                    operatorBalance
                );

                // Add it to the total TVL for this OD
                operatorTVL += operatorValues[j];

                // record token value of withdraw queue
                if (!withdrawQueueTokenBalanceRecorded) {
                    totalWithdrawalQueueValue += renzoOracle.lookupTokenValue(
                        collateralTokens[i],
                        collateralTokens[j].balanceOf(withdrawQueue)
                    );
                }

                unchecked {
                    ++j;
                }
            }

            // Get the value of native ETH staked for the OD
            uint256 operatorEthBalance = operatorDelegators[i].getStakedETHBalance();

            // Save it to the array for the OD
            operatorValues[operatorValues.length - 1] = operatorEthBalance;

            // Add it to the total TVL for this OD
            operatorTVL += operatorEthBalance;

            // Add it to the total TVL for the protocol
            totalTVL += operatorTVL;

            // Save the TVL for this OD
            operatorDelegatorTVLs[i] = operatorTVL;

            // Set withdrawQueueTokenBalanceRecorded flag to true
            withdrawQueueTokenBalanceRecorded = true;

            unchecked {
                ++i;
            }
        }

        // Get the value of native ETH held in the deposit queue and add it to the total TVL
        totalTVL += address(depositQueue).balance;

        // Add native ETH help in withdraw Queue and totalWithdrawalQueueValue to totalTVL
        totalTVL += (address(withdrawQueue).balance + totalWithdrawalQueueValue);

        return (operatorDelegatorTokenTVLs, operatorDelegatorTVLs, totalTVL);
    }

    function setPaused(bool _paused) external onlyDepositWithdrawPauserAdmin {
        paused = _paused;
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L121:123

```solidity
File: contracts/Rewards/RewardHandler.sol


    function initialize(IRoleManager _roleManager, address _rewardDestination) public initializer {
        __ReentrancyGuard_init();

        if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
        if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();

        roleManager = _roleManager;
        rewardDestination = _rewardDestination;

        emit RewardDestinationUpdated(_rewardDestination);
    }

    receive() external payable nonReentrant {
        _forwardETH();
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L52:54

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


    function initialize(
        IRoleManager _roleManager,
        IRestakeManager _restakeManager,
        IEzEthToken _ezETH,
        IRenzoOracle _renzoOracle,
        uint256 _coolDownPeriod,
        TokenWithdrawBuffer[] calldata _withdrawalBufferTarget
    ) external initializer {
        if (
            address(_roleManager) == address(0) ||
            address(_ezETH) == address(0) ||
            address(_renzoOracle) == address(0) ||
            address(_restakeManager) == address(0) ||
            _withdrawalBufferTarget.length == 0 ||
            _coolDownPeriod == 0
        ) revert InvalidZeroInput();

        __Pausable_init();

        roleManager = _roleManager;
        restakeManager = _restakeManager;
        ezETH = _ezETH;
        renzoOracle = _renzoOracle;
        coolDownPeriod = _coolDownPeriod;
        for (uint256 i = 0; i < _withdrawalBufferTarget.length; ) {
            if (
                _withdrawalBufferTarget[i].asset == address(0) ||
                _withdrawalBufferTarget[i].bufferAmount == 0
            ) revert InvalidZeroInput();
            withdrawalBufferTarget[_withdrawalBufferTarget[i].asset] = _withdrawalBufferTarget[i]
                .bufferAmount;
            unchecked {
                ++i;
            }
        }
    }

    function pause() external onlyWithdrawQueueAdmin {
        _pause();
    }

    function withdraw(uint256 _amount, address _assetOut) external nonReentrant {
        // check for 0 values
        if (_amount == 0 || _assetOut == address(0)) revert InvalidZeroInput();

        // check if provided assetOut is supported
        if (withdrawalBufferTarget[_assetOut] == 0) revert UnsupportedWithdrawAsset();

        // transfer ezETH tokens to this address
        IERC20(address(ezETH)).safeTransferFrom(msg.sender, address(this), _amount);

        // calculate totalTVL
        (, , uint256 totalTVL) = restakeManager.calculateTVLs();

        // Calculate amount to Redeem in ETH
        uint256 amountToRedeem = renzoOracle.calculateRedeemAmount(
            _amount,
            ezETH.totalSupply(),
            totalTVL
        );

        // update amount in claim asset, if claim asset is not ETH
        if (_assetOut != IS_NATIVE) {
            // Get ERC20 asset equivalent amount
            amountToRedeem = renzoOracle.lookupTokenAmountFromValue(
                IERC20(_assetOut),
                amountToRedeem
            );
        }

        // revert if amount to redeem is greater than withdrawBufferTarget
        if (amountToRedeem > getAvailableToWithdraw(_assetOut)) revert NotEnoughWithdrawBuffer();

        // increment the withdrawRequestNonce
        withdrawRequestNonce++;

        // add withdraw request for msg.sender
        withdrawRequests[msg.sender].push(
            WithdrawRequest(
                _assetOut,
                withdrawRequestNonce,
                amountToRedeem,
                _amount,
                block.timestamp
            )
        );

        // add redeem amount to claimReserve of claim asset
        claimReserve[_assetOut] += amountToRedeem;

        emit WithdrawRequestCreated(
            withdrawRequestNonce,
            msg.sender,
            _assetOut,
            amountToRedeem,
            _amount,
            withdrawRequests[msg.sender].length - 1
        );
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L206:263

```solidity
File: contracts/token/EzEthToken.sol


    function initialize(IRoleManager _roleManager) public initializer {
        if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();

        __ERC20_init("ezETH", "Renzo Restaked ETH");
        roleManager = _roleManager;
    }

    function mint(address to, uint256 amount) external onlyMinterBurner {
        _mint(to, amount);
    }

    function burn(address from, uint256 amount) external onlyMinterBurner {
        _burn(from, amount);
    }

    function setPaused(bool _paused) external onlyTokenAdmin {
        paused = _paused;
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L51:53

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


    function xReceive(
        bytes32 _transferId,
        uint256,
        address,
        address _originSender,
        uint32 _origin,
        bytes memory _callData
    ) external onlySource(_originSender, _origin) whenNotPaused returns (bytes memory) {
        (uint256 _price, uint256 _timestamp) = abi.decode(_callData, (uint256, uint256));
        xRenzoDeposit.updatePrice(_price, _timestamp);

        emit MessageReceived(_transferId, _origin, _originSender, _price, _timestamp);
    }

    function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {
        if (_newXRenzoBridgeL1 == address(0)) revert InvalidZeroInput();
        emit XRenzoBridgeL1Updated(_newXRenzoBridgeL1, xRenzoBridgeL1);
        xRenzoBridgeL1 = _newXRenzoBridgeL1;
    }

    function updateCCIPEthChainSelector(uint32 _newChainDomain) external onlyOwner {
        if (_newChainDomain == 0) revert InvalidZeroInput();
        emit ConnextEthChainDomainUpdated(_newChainDomain, connextEthChainDomain);
        connextEthChainDomain = _newChainDomain;
    }

    function unPause() external onlyOwner {
        _unpause();
    }

    function pause() external onlyOwner {
        _pause();
    }

    function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {
        if (_newXRenzoDeposit == address(0)) revert InvalidZeroInput();
        emit XRenzoDepositUpdated(_newXRenzoDeposit, address(xRenzoDeposit));
        xRenzoDeposit = IxRenzoDeposit(_newXRenzoDeposit);
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L130:134

```solidity
File: contracts/TimelockController.sol


    receive() external payable {}

    function supportsInterface(
        bytes4 interfaceId
    ) public view virtual override(IERC165, AccessControl) returns (bool) {
        return
            interfaceId == type(IERC1155Receiver).interfaceId ||
            super.supportsInterface(interfaceId);
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L142:148

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


    function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {
        if (_newXRenzoBridgeL1 == address(0)) revert InvalidZeroInput();
        emit XRenzoBridgeL1Updated(_newXRenzoBridgeL1, xRenzoBridgeL1);
        xRenzoBridgeL1 = _newXRenzoBridgeL1;
    }

    function updateCCIPEthChainSelector(uint64 _newChainSelector) external onlyOwner {
        if (_newChainSelector == 0) revert InvalidZeroInput();
        emit CCIPEthChainSelectorUpdated(_newChainSelector, ccipEthChainSelector);
        ccipEthChainSelector = _newChainSelector;
    }

    function unPause() external onlyOwner {
        _unpause();
    }

    function pause() external onlyOwner {
        _pause();
    }

    function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {
        if (_newXRenzoDeposit == address(0)) revert InvalidZeroInput();
        emit XRenzoDepositUpdated(_newXRenzoDeposit, address(xRenzoDeposit));
        xRenzoDeposit = IxRenzoDeposit(_newXRenzoDeposit);
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L134:138

```solidity
File: contracts/IRestakeManager.sol


    function depositETH() external payable;

    function stakeEthInOperatorDelegator(
        IOperatorDelegator operatorDelegator,
        bytes calldata pubkey,
        bytes calldata signature,
        bytes32 depositDataRoot
    ) external payable;

    function depositTokenRewardsFromProtocol(IERC20 _token, uint256 _amount) external;

    function calculateTVLs() external view returns (uint256[][] memory, uint256[] memory, uint256);

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/IRestakeManager.sol#L17:17

</details>

## NC043 - It is standard for all external and public functions to be override from an interface:

This is to ensure the whole API is extracted in an interface


<details>
<summary>Click to show 23 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


56          function bridgeTo(
57              address _to,
58              address _erc20,
59              address _remoteToken,
60              uint256 _amount,
61              uint32 _minGasLimit,
62              bytes calldata _extraData
63          ) external {
64              // Sanity check
65              if (_amount <= 0) {
66                  revert AmountLessThanZero();
67              }
68      
69              address xerc20 = IXERC20Registry(registry).getXERC20(_erc20);
70              address lockbox = IXERC20Registry(registry).getLockbox(xerc20);
71      
72              // Sanity check
73              if (xerc20 == address(0) || lockbox == address(0)) {
74                  revert InvalidAddress();
75              }
76      
77              // If using xERC20, the assumption is that the contract should be deployed at same address
78              // on both networks.
79              if (xerc20 != _remoteToken) {
80                  revert InvalidRemoteToken(_remoteToken);
81              }
82      
83              SafeERC20.safeTransferFrom(IERC20(_erc20), msg.sender, address(this), _amount);
84              SafeERC20.safeApprove(IERC20(_erc20), lockbox, _amount);
85              IXERC20Lockbox(lockbox).deposit(_amount);
86              SafeERC20.safeApprove(IERC20(xerc20), blastStandardBridge, _amount);
87              L1StandardBridge(blastStandardBridge).bridgeERC20To(
88                  xerc20,
89                  _remoteToken,
90                  _to,
91                  _amount,
92                  _minGasLimit,
93                  _extraData
94              );
95          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


70          function initialize(
71              IERC20 _ezETH,
72              IERC20 _xezETH,
73              IRestakeManager _restakeManager,
74              IERC20 _wETH,
75              IXERC20Lockbox _xezETHLockbox,
76              IConnext _connext,
77              IRouterClient _linkRouterClient,
78              IRateProvider _rateProvider,
79              LinkTokenInterface _linkToken,
80              IRoleManager _roleManager
81          ) public initializer {
82              // Verify non-zero addresses on inputs
83              if (
84                  address(_ezETH) == address(0) ||
85                  address(_xezETH) == address(0) ||
86                  address(_restakeManager) == address(0) ||
87                  address(_wETH) == address(0) ||
88                  address(_xezETHLockbox) == address(0) ||
89                  address(_connext) == address(0) ||
90                  address(_linkRouterClient) == address(0) ||
91                  address(_rateProvider) == address(0) ||
92                  address(_linkToken) == address(0) ||
93                  address(_roleManager) == address(0)
94              ) {
95                  revert InvalidZeroInput();
96              }
97      
98              // Verify all tokens have 18 decimals
99              uint8 decimals = IERC20MetadataUpgradeable(address(_ezETH)).decimals();
100             if (decimals != EXPECTED_DECIMALS) {
101                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
102             }
103             decimals = IERC20MetadataUpgradeable(address(_xezETH)).decimals();
104             if (decimals != EXPECTED_DECIMALS) {
105                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
106             }
107             decimals = IERC20MetadataUpgradeable(address(_wETH)).decimals();
108             if (decimals != EXPECTED_DECIMALS) {
109                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
110             }
111             decimals = IERC20MetadataUpgradeable(address(_linkToken)).decimals();
112             if (decimals != EXPECTED_DECIMALS) {
113                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
114             }
115     
116             // Save off inputs
117             ezETH = _ezETH;
118             xezETH = _xezETH;
119             restakeManager = _restakeManager;
120             wETH = _wETH;
121             xezETHLockbox = _xezETHLockbox;
122             connext = _connext;
123             linkRouterClient = _linkRouterClient;
124             rateProvider = _rateProvider;
125             linkToken = _linkToken;
126             roleManager = _roleManager;
127         }
139         function xReceive(
140             bytes32 _transferId,
141             uint256 _amount,
142             address _asset,
143             address _originSender,
144             uint32 _origin,
145             bytes memory
146         ) external nonReentrant returns (bytes memory) {
147             // Only allow incoming messages from the Connext contract
148             if (msg.sender != address(connext)) {
149                 revert InvalidSender(address(connext), msg.sender);
150             }
151     
152             // Check that the token received is wETH
153             if (_asset != address(wETH)) {
154                 revert InvalidTokenReceived();
155             }
156     
157             // Check that the amount sent is greater than 0
158             if (_amount == 0) {
159                 revert InvalidZeroInput();
160             }
161     
162             // Get the balance of ETH before the withdraw
163             uint256 ethBalanceBeforeWithdraw = address(this).balance;
164     
165             // Unwrap the WETH
166             IWeth(address(wETH)).withdraw(_amount);
167     
168             // Get the amount of ETH
169             uint256 ethAmount = address(this).balance - ethBalanceBeforeWithdraw;
170     
171             // Get the amonut of ezETH before the deposit
172             uint256 ezETHBalanceBeforeDeposit = ezETH.balanceOf(address(this));
173     
174             // Deposit it into Renzo RestakeManager
175             restakeManager.depositETH{ value: ethAmount }();
176     
177             // Get the amount of ezETH that was minted
178             uint256 ezETHAmount = ezETH.balanceOf(address(this)) - ezETHBalanceBeforeDeposit;
179     
180             // Approve the lockbox to spend the ezETH
181             ezETH.safeApprove(address(xezETHLockbox), ezETHAmount);
182     
183             // Get the xezETH balance before the deposit
184             uint256 xezETHBalanceBeforeDeposit = xezETH.balanceOf(address(this));
185     
186             // Send to the lockbox to be wrapped into xezETH
187             xezETHLockbox.deposit(ezETHAmount);
188     
189             // Get the amount of xezETH that was minted
190             uint256 xezETHAmount = xezETH.balanceOf(address(this)) - xezETHBalanceBeforeDeposit;
191     
192             // Burn it - it was already minted on the L2
193             IXERC20(address(xezETH)).burn(address(this), xezETHAmount);
194     
195             // Emit the event
196             emit EzETHMinted(_transferId, _amount, _origin, _originSender, ezETHAmount);
197     
198             // Return 0 for success
199             bytes memory returnData = new bytes(0);
200             return returnData;
201         }
210         function sendPrice(
211             CCIPDestinationParam[] calldata _destinationParam,
212             ConnextDestinationParam[] calldata _connextDestinationParam
213         ) external payable onlyPriceFeedSender nonReentrant {
214             // call getRate() to get the current price of ezETH
215             uint256 exchangeRate = rateProvider.getRate();
216             bytes memory _callData = abi.encode(exchangeRate, block.timestamp);
217             // send price feed to renzo CCIP receivers
218             for (uint256 i = 0; i < _destinationParam.length; ) {
219                 Client.EVM2AnyMessage memory evm2AnyMessage = Client.EVM2AnyMessage({
220                     receiver: abi.encode(_destinationParam[i]._renzoReceiver), // ABI-encoded xRenzoDepsot contract address
221                     data: _callData, // ABI-encoded ezETH exchange rate with Timestamp
222                     tokenAmounts: new Client.EVMTokenAmount[](0), // Empty array indicating no tokens are being sent
223                     extraArgs: Client._argsToBytes(
224                         // Additional arguments, setting gas limit
225                         Client.EVMExtraArgsV1({ gasLimit: 200_000 })
226                     ),
227                     // Set the feeToken  address, indicating LINK will be used for fees
228                     feeToken: address(linkToken)
229                 });
230     
231                 // Get the fee required to send the message
232                 uint256 fees = linkRouterClient.getFee(
233                     _destinationParam[i].destinationChainSelector,
234                     evm2AnyMessage
235                 );
236     
237                 if (fees > linkToken.balanceOf(address(this)))
238                     revert NotEnoughBalance(linkToken.balanceOf(address(this)), fees);
239     
240                 // approve the Router to transfer LINK tokens on contract's behalf. It will spend the fees in LINK
241                 linkToken.approve(address(linkRouterClient), fees);
242     
243                 // Send the message through the router and store the returned message ID
244                 bytes32 messageId = linkRouterClient.ccipSend(
245                     _destinationParam[i].destinationChainSelector,
246                     evm2AnyMessage
247                 );
248     
249                 // Emit an event with message details
250                 emit MessageSent(
251                     messageId,
252                     _destinationParam[i].destinationChainSelector,
253                     _destinationParam[i]._renzoReceiver,
254                     exchangeRate,
255                     address(linkToken),
256                     fees
257                 );
258                 unchecked {
259                     ++i;
260                 }
261             }
262     
263             // send price feed to renzo connext receiver
264             for (uint256 i = 0; i < _connextDestinationParam.length; ) {
265                 connext.xcall{ value: _connextDestinationParam[i].relayerFee }(
266                     _connextDestinationParam[i].destinationDomainId,
267                     _connextDestinationParam[i]._renzoReceiver,
268                     address(0),
269                     msg.sender,
270                     0,
271                     0,
272                     _callData
273                 );
274     
275                 emit ConnextMessageSent(
276                     _connextDestinationParam[i].destinationDomainId,
277                     _connextDestinationParam[i]._renzoReceiver,
278                     exchangeRate,
279                     _connextDestinationParam[i].relayerFee
280                 );
281     
282                 unchecked {
283                     ++i;
284                 }
285             }
286         }
294         function recoverNative(uint256 _amount, address _to) external onlyBridgeAdmin {
295             payable(_to).transfer(_amount);
296         }
305         function recoverERC20(address _token, uint256 _amount, address _to) external onlyBridgeAdmin {
306             IERC20(_token).safeTransfer(_to, _amount);
307         }
313         receive() external payable {}


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


23          function initialize(AggregatorV3Interface _oracle) public initializer {
24              // Initialize inherited classes
25              __Ownable_init();
26      
27              if (address(_oracle) == address(0)) revert InvalidZeroInput();
28      
29              // Verify that the pricing of the oracle less than or equal 18 decimals - pricing calculations will be off otherwise
30              if (_oracle.decimals() > 18) revert InvalidTokenDecimals(18, _oracle.decimals());
31      
32              oracle = _oracle;
33          }
36          function setOracleAddress(AggregatorV3Interface _oracleAddress) external onlyOwner {
37              if (address(_oracleAddress) == address(0)) revert InvalidZeroInput();
38              // Verify that the pricing of the oracle is less than or equal to 18 decimals - pricing calculations will be off otherwise
39              if (_oracleAddress.decimals() > 18)
40                  revert InvalidTokenDecimals(18, _oracleAddress.decimals());
41      
42              emit OracleAddressUpdated(address(_oracleAddress), address(oracle));
43              oracle = _oracleAddress;
44          }
50          function getMintRate() public view returns (uint256, uint256) {
51              (, int256 price, , uint256 timestamp, ) = oracle.latestRoundData();
52              if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();
53              // scale the price to have 18 decimals
54              uint256 _scaledPrice = (uint256(price)) * 10 ** (18 - oracle.decimals());
55              if (_scaledPrice < 1 ether) revert InvalidOraclePrice();
56              return (_scaledPrice, timestamp);
57          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


96          function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {
97              if (_newXRenzoBridgeL1 == address(0)) revert InvalidZeroInput();
98              emit XRenzoBridgeL1Updated(_newXRenzoBridgeL1, xRenzoBridgeL1);
99              xRenzoBridgeL1 = _newXRenzoBridgeL1;
100         }
107         function updateCCIPEthChainSelector(uint64 _newChainSelector) external onlyOwner {
108             if (_newChainSelector == 0) revert InvalidZeroInput();
109             emit CCIPEthChainSelectorUpdated(_newChainSelector, ccipEthChainSelector);
110             ccipEthChainSelector = _newChainSelector;
111         }
117         function unPause() external onlyOwner {
118             _unpause();
119         }
125         function pause() external onlyOwner {
126             _pause();
127         }
134         function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {
135             if (_newXRenzoDeposit == address(0)) revert InvalidZeroInput();
136             emit XRenzoDepositUpdated(_newXRenzoDeposit, address(xRenzoDeposit));
137             xRenzoDeposit = IxRenzoDeposit(_newXRenzoDeposit);
138         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


69          function xReceive(
70              bytes32 _transferId,
71              uint256,
72              address,
73              address _originSender,
74              uint32 _origin,
75              bytes memory _callData
76          ) external onlySource(_originSender, _origin) whenNotPaused returns (bytes memory) {
77              (uint256 _price, uint256 _timestamp) = abi.decode(_callData, (uint256, uint256));
78              xRenzoDeposit.updatePrice(_price, _timestamp);
79      
80              emit MessageReceived(_transferId, _origin, _originSender, _price, _timestamp);
81          }
92          function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {
93              if (_newXRenzoBridgeL1 == address(0)) revert InvalidZeroInput();
94              emit XRenzoBridgeL1Updated(_newXRenzoBridgeL1, xRenzoBridgeL1);
95              xRenzoBridgeL1 = _newXRenzoBridgeL1;
96          }
103         function updateCCIPEthChainSelector(uint32 _newChainDomain) external onlyOwner {
104             if (_newChainDomain == 0) revert InvalidZeroInput();
105             emit ConnextEthChainDomainUpdated(_newChainDomain, connextEthChainDomain);
106             connextEthChainDomain = _newChainDomain;
107         }
113         function unPause() external onlyOwner {
114             _unpause();
115         }
121         function pause() external onlyOwner {
122             _pause();
123         }
130         function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {
131             if (_newXRenzoDeposit == address(0)) revert InvalidZeroInput();
132             emit XRenzoDepositUpdated(_newXRenzoDeposit, address(xRenzoDeposit));
133             xRenzoDeposit = IxRenzoDeposit(_newXRenzoDeposit);
134         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


75          function initialize(
76              uint256 _currentPrice,
77              IERC20 _xezETH,
78              IERC20 _depositToken,
79              IERC20 _collateralToken,
80              IConnext _connext,
81              bytes32 _swapKey,
82              address _receiver,
83              uint32 _bridgeDestinationDomain,
84              address _bridgeTargetAddress,
85              IRenzoOracleL2 _oracle
86          ) public initializer {
87              // Initialize inherited classes
88              __Ownable_init();
89      
90              // Verify valid non zero values
91              if (
92                  _currentPrice == 0 ||
93                  address(_xezETH) == address(0) ||
94                  address(_depositToken) == address(0) ||
95                  address(_collateralToken) == address(0) ||
96                  address(_connext) == address(0) ||
97                  _swapKey == 0 ||
98                  _bridgeDestinationDomain == 0 ||
99                  _bridgeTargetAddress == address(0)
100             ) {
101                 revert InvalidZeroInput();
102             }
103     
104             // Verify all tokens have 18 decimals
105             uint8 decimals = IERC20MetadataUpgradeable(address(_depositToken)).decimals();
106             if (decimals != EXPECTED_DECIMALS) {
107                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
108             }
109             decimals = IERC20MetadataUpgradeable(address(_collateralToken)).decimals();
110             if (decimals != EXPECTED_DECIMALS) {
111                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
112             }
113             decimals = IERC20MetadataUpgradeable(address(_xezETH)).decimals();
114             if (decimals != EXPECTED_DECIMALS) {
115                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
116             }
117     
118             // Initialize the price and timestamp
119             lastPrice = _currentPrice;
120             lastPriceTimestamp = block.timestamp;
121     
122             // Set xezETH address
123             xezETH = _xezETH;
124     
125             // Set the depoist token
126             depositToken = _depositToken;
127     
128             // Set the collateral token
129             collateralToken = _collateralToken;
130     
131             // Set the connext contract
132             connext = _connext;
133     
134             // Set the swap key
135             swapKey = _swapKey;
136     
137             // Set receiver contract address
138             receiver = _receiver;
139             // Connext router fee is 5 basis points
140             bridgeRouterFeeBps = 5;
141     
142             // Set the bridge destination domain
143             bridgeDestinationDomain = _bridgeDestinationDomain;
144     
145             // Set the bridge target address
146             bridgeTargetAddress = _bridgeTargetAddress;
147     
148             // set oracle Price Feed struct
149             oracle = _oracle;
150     
151             // set bridge Fee Share 0.05% where 100 basis point = 1%
152             bridgeFeeShare = 5;
153     
154             //set sweep batch size to 32 ETH
155             sweepBatchSize = 32 ether;
156         }
168         function depositETH(
169             uint256 _minOut,
170             uint256 _deadline
171         ) external payable nonReentrant returns (uint256) {
172             if (msg.value == 0) {
173                 revert InvalidZeroInput();
174             }
175     
176             // Get the deposit token balance before
177             uint256 depositBalanceBefore = depositToken.balanceOf(address(this));
178     
179             // Wrap the deposit ETH to WETH
180             IWeth(address(depositToken)).deposit{ value: msg.value }();
181     
182             // Get the amount of tokens that were wrapped
183             uint256 wrappedAmount = depositToken.balanceOf(address(this)) - depositBalanceBefore;
184     
185             // Sanity check for 0
186             if (wrappedAmount == 0) {
187                 revert InvalidZeroOutput();
188             }
189     
190             return _deposit(wrappedAmount, _minOut, _deadline);
191         }
204         function deposit(
205             uint256 _amountIn,
206             uint256 _minOut,
207             uint256 _deadline
208         ) external nonReentrant returns (uint256) {
209             if (_amountIn == 0) {
210                 revert InvalidZeroInput();
211             }
212     
213             // Transfer deposit tokens from user to this contract
214             depositToken.safeTransferFrom(msg.sender, address(this), _amountIn);
215     
216             return _deposit(_amountIn, _minOut, _deadline);
217         }
277         function getBridgeFeeShare(uint256 _amountIn) public view returns (uint256) {
278             // deduct bridge Fee share
279             if (_amountIn < sweepBatchSize) {
280                 return (_amountIn * bridgeFeeShare) / FEE_BASIS;
281             } else {
282                 return (sweepBatchSize * bridgeFeeShare) / FEE_BASIS;
283             }
284         }
289         function getMintRate() public view returns (uint256, uint256) {
290             // revert if PriceFeedNotAvailable
291             if (receiver == address(0) && address(oracle) == address(0)) revert PriceFeedNotAvailable();
292             if (address(oracle) != address(0)) {
293                 (uint256 oraclePrice, uint256 oracleTimestamp) = oracle.getMintRate();
294                 return
295                     oracleTimestamp > lastPriceTimestamp
296                         ? (oraclePrice, oracleTimestamp)
297                         : (lastPrice, lastPriceTimestamp);
298             } else {
299                 return (lastPrice, lastPriceTimestamp);
300             }
301         }
310         function updatePrice(uint256 _price, uint256 _timestamp) external override {
311             if (msg.sender != receiver) revert InvalidSender(receiver, msg.sender);
312             _updatePrice(_price, _timestamp);
313         }
320         function updatePriceByOwner(uint256 price) external onlyOwner {
321             return _updatePrice(price, block.timestamp);
322         }
414         function sweep() public payable nonReentrant {
415             // Verify the caller is whitelisted
416             if (!allowedBridgeSweepers[msg.sender]) {
417                 revert UnauthorizedBridgeSweeper();
418             }
419     
420             // Get the balance of nextWETH in the contract
421             uint256 balance = collateralToken.balanceOf(address(this));
422     
423             // If there is no balance, return
424             if (balance == 0) {
425                 revert InvalidZeroOutput();
426             }
427     
428             // Approve it to the connext contract
429             collateralToken.safeApprove(address(connext), balance);
430     
431             // Need to send some calldata so it triggers xReceive on the target
432             bytes memory bridgeCallData = abi.encode(balance);
433     
434             connext.xcall{ value: msg.value }(
435                 bridgeDestinationDomain,
436                 bridgeTargetAddress,
437                 address(collateralToken),
438                 msg.sender,
439                 balance,
440                 0, // Asset is already nextWETH, so no slippage will be incurred
441                 bridgeCallData
442             );
443     
444             // send collected bridge fee to sweeper
445             _recoverBridgeFee();
446     
447             // Emit the event
448             emit BridgeSwept(bridgeDestinationDomain, bridgeTargetAddress, msg.sender, balance);
449         }
456         function getRate() external view override returns (uint256) {
457             return lastPrice;
458         }
466         function setAllowedBridgeSweeper(address _sweeper, bool _allowed) external onlyOwner {
467             allowedBridgeSweepers[_sweeper] = _allowed;
468     
469             emit BridgeSweeperAddressUpdated(_sweeper, _allowed);
470         }
478         function recoverNative(uint256 _amount, address _to) external onlyOwner {
479             payable(_to).transfer(_amount);
480         }
489         function recoverERC20(address _token, uint256 _amount, address _to) external onlyOwner {
490             IERC20(_token).safeTransfer(_to, _amount);
491         }
501         function setOraclePriceFeed(IRenzoOracleL2 _oracle) external onlyOwner {
502             emit OraclePriceFeedUpdated(address(_oracle), address(oracle));
503             oracle = _oracle;
504         }
511         function setReceiverPriceFeed(address _receiver) external onlyOwner {
512             emit ReceiverPriceFeedUpdated(_receiver, receiver);
513             receiver = _receiver;
514         }
521         function updateBridgeFeeShare(uint256 _newShare) external onlyOwner {
522             if (_newShare > 100) revert InvalidBridgeFeeShare(_newShare);
523             emit BridgeFeeShareUpdated(bridgeFeeShare, _newShare);
524             bridgeFeeShare = _newShare;
525         }
532         function updateSweepBatchSize(uint256 _newBatchSize) external onlyOwner {
533             if (_newBatchSize < 32 ether) revert InvalidSweepBatchSize(_newBatchSize);
534             emit SweepBatchSizeUpdated(sweepBatchSize, _newBatchSize);
535             sweepBatchSize = _newBatchSize;
536         }
542         receive() external payable {}


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


61          function initialize(
62              string memory _name,
63              string memory _symbol,
64              address _factory
65          ) public initializer {
66              __XERC20_init(_name, _symbol, _factory);
67          }
96          function mint(address _user, uint256 _amount) public virtual {
97              _mintWithCaller(msg.sender, _user, _amount);
98          }
107         function burn(address _user, uint256 _amount) public virtual {
108             if (msg.sender != _user) {
109                 _spendAllowance(_user, msg.sender, _amount);
110             }
111     
112             _burnWithCaller(msg.sender, _user, _amount);
113         }
121         function setLockbox(address _lockbox) public {
122             if (msg.sender != FACTORY) revert IXERC20_NotFactory();
123             lockbox = _lockbox;
124     
125             emit LockboxSet(_lockbox);
126         }
135         function setLimits(
136             address _bridge,
137             uint256 _mintingLimit,
138             uint256 _burningLimit
139         ) external onlyOwner {
140             _changeMinterLimit(_bridge, _mintingLimit);
141             _changeBurnerLimit(_bridge, _burningLimit);
142             emit BridgeLimitsSet(_mintingLimit, _burningLimit, _bridge);
143         }
152         function mintingMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
153             _limit = bridges[_bridge].minterParams.maxLimit;
154         }
163         function burningMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
164             _limit = bridges[_bridge].burnerParams.maxLimit;
165         }
174         function mintingCurrentLimitOf(address _bridge) public view returns (uint256 _limit) {
175             _limit = _getCurrentLimit(
176                 bridges[_bridge].minterParams.currentLimit,
177                 bridges[_bridge].minterParams.maxLimit,
178                 bridges[_bridge].minterParams.timestamp,
179                 bridges[_bridge].minterParams.ratePerSecond
180             );
181         }
190         function burningCurrentLimitOf(address _bridge) public view returns (uint256 _limit) {
191             _limit = _getCurrentLimit(
192                 bridges[_bridge].burnerParams.currentLimit,
193                 bridges[_bridge].burnerParams.maxLimit,
194                 bridges[_bridge].burnerParams.timestamp,
195                 bridges[_bridge].burnerParams.ratePerSecond
196             );
197         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


54          function initialize(
55              address _lockboxImplementation,
56              address _xerc20Implementation
57          ) public initializer {
58              lockboxImplementation = _lockboxImplementation;
59              xerc20Implementation = _xerc20Implementation;
60          }
74          function deployXERC20(
75              string memory _name,
76              string memory _symbol,
77              uint256[] memory _minterLimits,
78              uint256[] memory _burnerLimits,
79              address[] memory _bridges,
80              address _proxyAdmin
81          ) external returns (address _xerc20) {
82              _xerc20 = _deployXERC20(
83                  _name,
84                  _symbol,
85                  _minterLimits,
86                  _burnerLimits,
87                  _bridges,
88                  _proxyAdmin
89              );
90      
91              emit XERC20Deployed(_xerc20);
92          }
105         function deployLockbox(
106             address _xerc20,
107             address _baseToken,
108             bool _isNative,
109             address _proxyAdmin
110         ) external returns (address payable _lockbox) {
111             if ((_baseToken == address(0) && !_isNative) || (_isNative && _baseToken != address(0))) {
112                 revert IXERC20Factory_BadTokenAddress();
113             }
114     
115             if (XERC20(_xerc20).owner() != msg.sender) revert IXERC20Factory_NotOwner();
116             if (_lockboxRegistry[_xerc20] != address(0)) revert IXERC20Factory_LockboxAlreadyDeployed();
117     
118             _lockbox = _deployLockbox(_xerc20, _baseToken, _isNative, _proxyAdmin);
119     
120             emit LockboxDeployed(_lockbox);
121         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


44          function initialize(address _xerc20, address _erc20, bool _isNative) public initializer {
45              XERC20 = IXERC20(_xerc20);
46              ERC20 = IERC20(_erc20);
47              IS_NATIVE = _isNative;
48          }
54          function depositNative() public payable {
55              if (!IS_NATIVE) revert IXERC20Lockbox_NotNative();
56      
57              _deposit(msg.sender, msg.value);
58          }
66          function deposit(uint256 _amount) external {
67              if (IS_NATIVE) revert IXERC20Lockbox_Native();
68      
69              _deposit(msg.sender, _amount);
70          }
79          function depositTo(address _to, uint256 _amount) external {
80              if (IS_NATIVE) revert IXERC20Lockbox_Native();
81      
82              _deposit(_to, _amount);
83          }
91          function depositNativeTo(address _to) public payable {
92              if (!IS_NATIVE) revert IXERC20Lockbox_NotNative();
93      
94              _deposit(_to, msg.value);
95          }
103         function withdraw(uint256 _amount) external {
104             _withdraw(msg.sender, _amount);
105         }
114         function withdrawTo(address _to, uint256 _amount) external {
115             _withdraw(_to, _amount);
116         }
157         receive() external payable {
158             depositNative();
159         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


35          function initialize(
36              string memory _name,
37              string memory _symbol,
38              address _factory,
39              address _l1Token,
40              address _optimismBridge
41          ) public initializer {
42              __ERC165_init();
43              __XERC20_init(_name, _symbol, _factory);
44              l1Token = _l1Token;
45              optimismBridge = _optimismBridge;
46          }
48          function supportsInterface(
49              bytes4 interfaceId
50          ) public view override(ERC165Upgradeable) returns (bool) {
51              return
52                  interfaceId == type(IOptimismMintableERC20).interfaceId ||
53                  super.supportsInterface(interfaceId);
54          }
56          function remoteToken() public view override returns (address) {
57              return l1Token;
58          }
60          function bridge() public view override returns (address) {
61              return optimismBridge;
62          }
64          function mint(address _to, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
65              XERC20.mint(_to, _amount);
66          }
68          function burn(address _from, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
69              XERC20.burn(_from, _amount);
70          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


37          function deployOptimismMintableXERC20(
38              string memory _name,
39              string memory _symbol,
40              uint256[] memory _minterLimits,
41              uint256[] memory _burnerLimits,
42              address[] memory _bridges,
43              address _proxyAdmin,
44              address _l1Token
45          ) public returns (address _xerc20) {
46              _xerc20 = _deployOptimismMintableXERC20(
47                  _name,
48                  _symbol,
49                  _minterLimits,
50                  _burnerLimits,
51                  _bridges,
52                  _proxyAdmin,
53                  _l1Token
54              );
55      
56              emit XERC20Deployed(_xerc20);
57          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


74          function initialize(
75              IRoleManager _roleManager,
76              IStrategyManager _strategyManager,
77              IRestakeManager _restakeManager,
78              IDelegationManager _delegationManager,
79              IEigenPodManager _eigenPodManager
80          ) external initializer {
81              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
82              if (address(_strategyManager) == address(0x0)) revert InvalidZeroInput();
83              if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
84              if (address(_delegationManager) == address(0x0)) revert InvalidZeroInput();
85              if (address(_eigenPodManager) == address(0x0)) revert InvalidZeroInput();
86      
87              __ReentrancyGuard_init();
88      
89              roleManager = _roleManager;
90              strategyManager = _strategyManager;
91              restakeManager = _restakeManager;
92              delegationManager = _delegationManager;
93              eigenPodManager = _eigenPodManager;
94      
95              // Deploy new EigenPod
96              eigenPod = IEigenPod(eigenPodManager.createPod());
97          }
101         function activateRestaking() external nonReentrant onlyNativeEthRestakeAdmin {
102             eigenPod.activateRestaking();
103         }
106         function setTokenStrategy(
107             IERC20 _token,
108             IStrategy _strategy
109         ) external nonReentrant onlyOperatorDelegatorAdmin {
110             if (address(_token) == address(0x0)) revert InvalidZeroInput();
111     
112             tokenStrategyMapping[_token] = _strategy;
113             emit TokenStrategyUpdated(_token, _strategy);
114         }
117         function setDelegateAddress(
118             address _delegateAddress,
119             ISignatureUtils.SignatureWithExpiry memory approverSignatureAndExpiry,
120             bytes32 approverSalt
121         ) external nonReentrant onlyOperatorDelegatorAdmin {
122             if (address(_delegateAddress) == address(0x0)) revert InvalidZeroInput();
123             if (address(delegateAddress) != address(0x0)) revert DelegateAddressAlreadySet();
124     
125             delegateAddress = _delegateAddress;
126     
127             delegationManager.delegateTo(delegateAddress, approverSignatureAndExpiry, approverSalt);
128     
129             emit DelegationAddressUpdated(_delegateAddress);
130         }
132         function setBaseGasAmountSpent(
133             uint256 _baseGasAmountSpent
134         ) external nonReentrant onlyOperatorDelegatorAdmin {
135             if (_baseGasAmountSpent == 0) revert InvalidZeroInput();
136             emit BaseGasAmountSpentUpdated(baseGasAmountSpent, _baseGasAmountSpent);
137             baseGasAmountSpent = _baseGasAmountSpent;
138         }
143         function deposit(
144             IERC20 token,
145             uint256 tokenAmount
146         ) external nonReentrant onlyRestakeManager returns (uint256 shares) {
147             if (address(tokenStrategyMapping[token]) == address(0x0) || tokenAmount == 0)
148                 revert InvalidZeroInput();
149     
150             // Move the tokens into this contract
151             token.safeTransferFrom(msg.sender, address(this), tokenAmount);
152     
153             return _deposit(token, tokenAmount);
154         }
172         function getStrategyIndex(IStrategy _strategy) public view returns (uint256) {
173             // Get the length of the strategy list for this contract
174             uint256 strategyLength = strategyManager.stakerStrategyListLength(address(this));
175     
176             for (uint256 i = 0; i < strategyLength; i++) {
177                 if (strategyManager.stakerStrategyList(address(this), i) == _strategy) {
178                     return i;
179                 }
180             }
181     
182             // Not found
183             revert NotFound();
184         }
193         function queueWithdrawals(
194             IERC20[] calldata tokens,
195             uint256[] calldata tokenAmounts
196         ) external nonReentrant onlyNativeEthRestakeAdmin returns (bytes32) {
197             // record gas spent
198             uint256 gasBefore = gasleft();
199             if (tokens.length != tokenAmounts.length) revert MismatchedArrayLengths();
200             IDelegationManager.QueuedWithdrawalParams[]
201                 memory queuedWithdrawalParams = new IDelegationManager.QueuedWithdrawalParams[](1);
202             // set strategies legth for 0th index only
203             queuedWithdrawalParams[0].strategies = new IStrategy[](tokens.length);
204             queuedWithdrawalParams[0].shares = new uint256[](tokens.length);
205     
206             // Save the nonce before starting the withdrawal
207             uint96 nonce = uint96(delegationManager.cumulativeWithdrawalsQueued(address(this)));
208     
209             for (uint256 i; i < tokens.length; ) {
210                 if (address(tokens[i]) == IS_NATIVE) {
211                     // set beaconChainEthStrategy for ETH
212                     queuedWithdrawalParams[0].strategies[i] = eigenPodManager.beaconChainETHStrategy();
213     
214                     // set shares for ETH
215                     queuedWithdrawalParams[0].shares[i] = tokenAmounts[i];
216                 } else {
217                     if (address(tokenStrategyMapping[tokens[i]]) == address(0))
218                         revert InvalidZeroInput();
219     
220                     // set the strategy of the token
221                     queuedWithdrawalParams[0].strategies[i] = tokenStrategyMapping[tokens[i]];
222     
223                     // set the equivalent shares for tokenAmount
224                     queuedWithdrawalParams[0].shares[i] = tokenStrategyMapping[tokens[i]]
225                         .underlyingToSharesView(tokenAmounts[i]);
226                 }
227     
228                 // set withdrawer as this contract address
229                 queuedWithdrawalParams[0].withdrawer = address(this);
230     
231                 // track shares of tokens withdraw for TVL
232                 queuedShares[address(tokens[i])] += queuedWithdrawalParams[0].shares[i];
233                 unchecked {
234                     ++i;
235                 }
236             }
237     
238             // queue withdrawal in EigenLayer
239             bytes32 withdrawalRoot = delegationManager.queueWithdrawals(queuedWithdrawalParams)[0];
240             // Emit the withdrawal started event
241             emit WithdrawStarted(
242                 withdrawalRoot,
243                 address(this),
244                 delegateAddress,
245                 address(this),
246                 nonce,
247                 block.number,
248                 queuedWithdrawalParams[0].strategies,
249                 queuedWithdrawalParams[0].shares
250             );
251     
252             // update the gas spent for RestakeAdmin
253             _recordGas(gasBefore);
254     
255             return withdrawalRoot;
256         }
265         function completeQueuedWithdrawal(
266             IDelegationManager.Withdrawal calldata withdrawal,
267             IERC20[] calldata tokens,
268             uint256 middlewareTimesIndex
269         ) external nonReentrant onlyNativeEthRestakeAdmin {
270             uint256 gasBefore = gasleft();
271             if (tokens.length != withdrawal.strategies.length) revert MismatchedArrayLengths();
272     
273             // complete the queued withdrawal from EigenLayer with receiveAsToken set to true
274             delegationManager.completeQueuedWithdrawal(withdrawal, tokens, middlewareTimesIndex, true);
275     
276             IWithdrawQueue withdrawQueue = restakeManager.depositQueue().withdrawQueue();
277             for (uint256 i; i < tokens.length; ) {
278                 if (address(tokens[i]) == address(0)) revert InvalidZeroInput();
279     
280                 // deduct queued shares for tracking TVL
281                 queuedShares[address(tokens[i])] -= withdrawal.shares[i];
282     
283                 // check if token is not Native ETH
284                 if (address(tokens[i]) != IS_NATIVE) {
285                     // Check the withdraw buffer and fill if below buffer target
286                     uint256 bufferToFill = withdrawQueue.getBufferDeficit(address(tokens[i]));
287     
288                     // get balance of this contract
289                     uint256 balanceOfToken = tokens[i].balanceOf(address(this));
290                     if (bufferToFill > 0) {
291                         bufferToFill = (balanceOfToken <= bufferToFill) ? balanceOfToken : bufferToFill;
292     
293                         // update amount to send to the operator Delegator
294                         balanceOfToken -= bufferToFill;
295     
296                         // safe Approve for depositQueue
297                         tokens[i].safeApprove(address(restakeManager.depositQueue()), bufferToFill);
298     
299                         // fill Withdraw Buffer via depositQueue
300                         restakeManager.depositQueue().fillERC20withdrawBuffer(
301                             address(tokens[i]),
302                             bufferToFill
303                         );
304                     }
305     
306                     // Deposit remaining tokens back to eigenLayer
307                     if (balanceOfToken > 0) {
308                         _deposit(tokens[i], balanceOfToken);
309                     }
310                 }
311                 unchecked {
312                     ++i;
313                 }
314             }
315     
316             // emits the Withdraw Completed event with withdrawalRoot
317             emit WithdrawCompleted(
318                 delegationManager.calculateWithdrawalRoot(withdrawal),
319                 withdrawal.strategies,
320                 withdrawal.shares
321             );
322             // record current spent gas
323             _recordGas(gasBefore);
324         }
327         function getTokenBalanceFromStrategy(IERC20 token) external view returns (uint256) {
328             return
329                 queuedShares[address(this)] == 0
330                     ? tokenStrategyMapping[token].userUnderlyingView(address(this))
331                     : tokenStrategyMapping[token].userUnderlyingView(address(this)) +
332                         tokenStrategyMapping[token].sharesToUnderlyingView(
333                             queuedShares[address(token)]
334                         );
335         }
338         function getStakedETHBalance() external view returns (uint256) {
339             // accounts for current podOwner shares + stakedButNotVerified ETH + queued withdraw shares
340             int256 podOwnerShares = eigenPodManager.podOwnerShares(address(this));
341             return
342                 podOwnerShares < 0
343                     ? queuedShares[IS_NATIVE] + stakedButNotVerifiedEth - uint256(-podOwnerShares)
344                     : queuedShares[IS_NATIVE] + stakedButNotVerifiedEth + uint256(podOwnerShares);
345         }
349         function stakeEth(
350             bytes calldata pubkey,
351             bytes calldata signature,
352             bytes32 depositDataRoot
353         ) external payable onlyRestakeManager {
354             // Call the stake function in the EigenPodManager
355             eigenPodManager.stake{ value: msg.value }(pubkey, signature, depositDataRoot);
356     
357             // Increment the staked but not verified ETH
358             stakedButNotVerifiedEth += msg.value;
359         }
364         function verifyWithdrawalCredentials(
365             uint64 oracleTimestamp,
366             BeaconChainProofs.StateRootProof calldata stateRootProof,
367             uint40[] calldata validatorIndices,
368             bytes[] calldata withdrawalCredentialProofs,
369             bytes32[][] calldata validatorFields
370         ) external onlyNativeEthRestakeAdmin {
371             uint256 gasBefore = gasleft();
372             eigenPod.verifyWithdrawalCredentials(
373                 oracleTimestamp,
374                 stateRootProof,
375                 validatorIndices,
376                 withdrawalCredentialProofs,
377                 validatorFields
378             );
379     
380             // Decrement the staked but not verified ETH
381             for (uint256 i = 0; i < validatorFields.length; ) {
382                 uint64 validatorCurrentBalanceGwei = BeaconChainProofs.getEffectiveBalanceGwei(
383                     validatorFields[i]
384                 );
385                 stakedButNotVerifiedEth -= (validatorCurrentBalanceGwei * GWEI_TO_WEI);
386                 unchecked {
387                     ++i;
388                 }
389             }
390             // update the gas spent for RestakeAdmin
391             _recordGas(gasBefore);
392         }
405         function verifyAndProcessWithdrawals(
406             uint64 oracleTimestamp,
407             BeaconChainProofs.StateRootProof calldata stateRootProof,
408             BeaconChainProofs.WithdrawalProof[] calldata withdrawalProofs,
409             bytes[] calldata validatorFieldsProofs,
410             bytes32[][] calldata validatorFields,
411             bytes32[][] calldata withdrawalFields
412         ) external onlyNativeEthRestakeAdmin {
413             uint256 gasBefore = gasleft();
414             eigenPod.verifyAndProcessWithdrawals(
415                 oracleTimestamp,
416                 stateRootProof,
417                 withdrawalProofs,
418                 validatorFieldsProofs,
419                 validatorFields,
420                 withdrawalFields
421             );
422             // update the gas spent for RestakeAdmin
423             _recordGas(gasBefore);
424         }
432         function withdrawNonBeaconChainETHBalanceWei(
433             address recipient,
434             uint256 amountToWithdraw
435         ) external onlyNativeEthRestakeAdmin {
436             eigenPod.withdrawNonBeaconChainETHBalanceWei(recipient, amountToWithdraw);
437         }
446         function recoverTokens(
447             IERC20[] memory tokenList,
448             uint256[] memory amountsToWithdraw,
449             address recipient
450         ) external onlyNativeEthRestakeAdmin {
451             eigenPod.recoverTokens(tokenList, amountsToWithdraw, recipient);
452         }
459         function startDelayedWithdrawUnstakedETH() external onlyNativeEthRestakeAdmin {
460             // Call the start delayed withdraw function in the EigenPodManager
461             // This will queue up a delayed withdrawal that will be sent back to this address after the timeout
462             eigenPod.withdrawBeforeRestaking();
463         }
501         receive() external payable nonReentrant {
502             // check if sender contract is EigenPod. forward full withdrawal eth received
503             if (msg.sender == address(eigenPod)) {
504                 restakeManager.depositQueue().forwardFullWithdrawalETH{ value: msg.value }();
505             } else {
506                 // considered as protocol reward
507                 uint256 gasRefunded = 0;
508                 uint256 remainingAmount = msg.value;
509                 if (adminGasSpentInWei[tx.origin] > 0) {
510                     gasRefunded = _refundGas();
511                     // update the remaining amount
512                     remainingAmount -= gasRefunded;
513                     // If no funds left, return
514                     if (remainingAmount == 0) {
515                         return;
516                     }
517                 }
518                 // Forward remaining balance to the deposit queue
519                 address destination = address(restakeManager.depositQueue());
520                 (bool success, ) = destination.call{ value: remainingAmount }("");
521                 if (!success) revert TransferFailed();
522     
523                 emit RewardsForwarded(destination, remainingAmount);
524             }
525         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


74          function initialize(IRoleManager _roleManager) public initializer {
75              __ReentrancyGuard_init();
76      
77              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
78      
79              roleManager = _roleManager;
80          }
87          function setWithdrawQueue(IWithdrawQueue _withdrawQueue) external onlyRestakeManagerAdmin {
88              if (address(_withdrawQueue) == address(0)) revert InvalidZeroInput();
89              emit WithdrawQueueUpdated(address(withdrawQueue), address(_withdrawQueue));
90              withdrawQueue = _withdrawQueue;
91          }
93          function setFeeConfig(
94              address _feeAddress,
95              uint256 _feeBasisPoints
96          ) external onlyRestakeManagerAdmin {
97              // Verify address is set if basis points are non-zero
98              if (_feeBasisPoints > 0) {
99                  if (_feeAddress == address(0x0)) revert InvalidZeroInput();
100             }
101     
102             // Verify basis points are not over 100%
103             if (_feeBasisPoints > 10000) revert OverMaxBasisPoints();
104     
105             feeAddress = _feeAddress;
106             feeBasisPoints = _feeBasisPoints;
107     
108             emit FeeConfigUpdated(_feeAddress, _feeBasisPoints);
109         }
112         function setRestakeManager(IRestakeManager _restakeManager) external onlyRestakeManagerAdmin {
113             if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
114     
115             restakeManager = _restakeManager;
116     
117             emit RestakeManagerUpdated(_restakeManager);
118         }
123         function depositETHFromProtocol() external payable onlyRestakeManager {
124             _checkAndFillETHWithdrawBuffer(msg.value);
125             emit ETHDepositedFromProtocol(msg.value);
126         }
134         function fillERC20withdrawBuffer(
135             address _asset,
136             uint256 _amount
137         ) external nonReentrant onlyRestakeManager {
138             if (_amount == 0 || _asset == address(0)) revert InvalidZeroInput();
139             // safeTransfer from restake manager to this address
140             IERC20(_asset).safeTransferFrom(msg.sender, address(this), _amount);
141             // approve the token amount for withdraw queue
142             IERC20(_asset).safeApprove(address(withdrawQueue), _amount);
143             // call the withdraw queue to fill up the buffer
144             withdrawQueue.fillERC20WithdrawBuffer(_asset, _amount);
145         }
152         function forwardFullWithdrawalETH() external payable nonReentrant {
153             // Check and fill ETH withdraw buffer if required
154             _checkAndFillETHWithdrawBuffer(msg.value);
155             emit FullWithdrawalETHReceived(msg.value);
156         }
163         receive() external payable nonReentrant {
164             uint256 feeAmount = 0;
165             // Take protocol cut of rewards if enabled
166             if (feeAddress != address(0x0) && feeBasisPoints > 0) {
167                 feeAmount = (msg.value * feeBasisPoints) / 10000;
168                 (bool success, ) = feeAddress.call{ value: feeAmount }("");
169                 if (!success) revert TransferFailed();
170     
171                 emit ProtocolFeesPaid(IERC20(address(0x0)), feeAmount, feeAddress);
172             }
173             // update remaining rewards
174             uint256 remainingRewards = msg.value - feeAmount;
175             // Check and fill ETH withdraw buffer if required
176             _checkAndFillETHWithdrawBuffer(remainingRewards);
177     
178             // Add to the total earned
179             totalEarned[address(0x0)] = totalEarned[address(0x0)] + remainingRewards;
180     
181             // Emit the rewards event
182             emit RewardsDeposited(IERC20(address(0x0)), remainingRewards);
183         }
187         function stakeEthFromQueue(
188             IOperatorDelegator operatorDelegator,
189             bytes calldata pubkey,
190             bytes calldata signature,
191             bytes32 depositDataRoot
192         ) external onlyNativeEthRestakeAdmin {
193             uint256 gasBefore = gasleft();
194             // Send the ETH and the params through to the restake manager
195             restakeManager.stakeEthInOperatorDelegator{ value: 32 ether }(
196                 operatorDelegator,
197                 pubkey,
198                 signature,
199                 depositDataRoot
200             );
201     
202             emit ETHStakedFromQueue(operatorDelegator, pubkey, 32 ether, address(this).balance);
203     
204             // Refund the gas to the Admin address if enough ETH available
205             _refundGas(gasBefore);
206         }
211         function stakeEthFromQueueMulti(
212             IOperatorDelegator[] calldata operatorDelegators,
213             bytes[] calldata pubkeys,
214             bytes[] calldata signatures,
215             bytes32[] calldata depositDataRoots
216         ) external onlyNativeEthRestakeAdmin nonReentrant {
217             uint256 gasBefore = gasleft();
218             // Verify all arrays are the same length
219             if (
220                 operatorDelegators.length != pubkeys.length ||
221                 operatorDelegators.length != signatures.length ||
222                 operatorDelegators.length != depositDataRoots.length
223             ) revert MismatchedArrayLengths();
224     
225             // Iterate through the arrays and stake each one
226             uint256 arrayLength = operatorDelegators.length;
227             for (uint256 i = 0; i < arrayLength; ) {
228                 // Send the ETH and the params through to the restake manager
229                 restakeManager.stakeEthInOperatorDelegator{ value: 32 ether }(
230                     operatorDelegators[i],
231                     pubkeys[i],
232                     signatures[i],
233                     depositDataRoots[i]
234                 );
235     
236                 emit ETHStakedFromQueue(
237                     operatorDelegators[i],
238                     pubkeys[i],
239                     32 ether,
240                     address(this).balance
241                 );
242     
243                 unchecked {
244                     ++i;
245                 }
246             }
247     
248             // Refund the gas to the Admin address if enough ETH available
249             _refundGas(gasBefore);
250         }
254         function sweepERC20(IERC20 token) external onlyERC20RewardsAdmin {
255             uint256 balance = IERC20(token).balanceOf(address(this));
256             if (balance > 0) {
257                 uint256 feeAmount = 0;
258     
259                 // Sweep fees if configured
260                 if (feeAddress != address(0x0) && feeBasisPoints > 0) {
261                     feeAmount = (balance * feeBasisPoints) / 10000;
262                     IERC20(token).safeTransfer(feeAddress, feeAmount);
263     
264                     emit ProtocolFeesPaid(token, feeAmount, feeAddress);
265                 }
266     
267                 // Approve and deposit the rewards
268                 token.approve(address(restakeManager), balance - feeAmount);
269                 restakeManager.depositTokenRewardsFromProtocol(token, balance - feeAmount);
270     
271                 // Add to the total earned
272                 totalEarned[address(token)] = totalEarned[address(token)] + balance - feeAmount;
273     
274                 // Emit the rewards event
275                 emit RewardsDeposited(IERC20(address(token)), balance - feeAmount);
276             }
277         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


23          function initialize(IStakedTokenV2 _wBETHToken) public initializer {
24              if (address(_wBETHToken) == address(0x0)) revert InvalidZeroInput();
25      
26              wBETHToken = _wBETHToken;
27          }
29          function decimals() external pure returns (uint8) {
30              return 18;
31          }
33          function description() external pure returns (string memory) {
34              return "wBETH Chainlink Shim";
35          }
37          function version() external pure returns (uint256) {
38              return 1;
39          }
42          function getRoundData(uint80) external pure returns (uint80, int256, uint256, uint256, uint80) {
43              revert NotImplemented();
44          }
46          function latestRoundData()
47              external
48              view
49              returns (
50                  uint80 roundId,
51                  int256 answer,
52                  uint256 startedAt,
53                  uint256 updatedAt,
54                  uint80 answeredInRound
55              )
56          {
57              return _getWBETHData();
58          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


23          function initialize(IMethStaking _methStaking) public initializer {
24              if (address(_methStaking) == address(0x0)) revert InvalidZeroInput();
25      
26              methStaking = _methStaking;
27          }
29          function decimals() external pure returns (uint8) {
30              return 18;
31          }
33          function description() external pure returns (string memory) {
34              return "METH Chainlink Shim";
35          }
37          function version() external pure returns (uint256) {
38              return 1;
39          }
42          function getRoundData(uint80) external pure returns (uint80, int256, uint256, uint256, uint80) {
43              revert NotImplemented();
44          }
46          function latestRoundData()
47              external
48              view
49              returns (
50                  uint80 roundId,
51                  int256 answer,
52                  uint256 startedAt,
53                  uint256 updatedAt,
54                  uint80 answeredInRound
55              )
56          {
57              return _getMETHData();
58          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


44          function initialize(IRoleManager _roleManager) public initializer {
45              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
46      
47              __ReentrancyGuard_init();
48      
49              roleManager = _roleManager;
50          }
54          function setOracleAddress(
55              IERC20 _token,
56              AggregatorV3Interface _oracleAddress
57          ) external nonReentrant onlyOracleAdmin {
58              if (address(_token) == address(0x0)) revert InvalidZeroInput();
59      
60              // Verify that the pricing of the oracle is 18 decimals - pricing calculations will be off otherwise
61              if (_oracleAddress.decimals() != 18)
62                  revert InvalidTokenDecimals(18, _oracleAddress.decimals());
63      
64              tokenOracleLookup[_token] = _oracleAddress;
65              emit OracleAddressUpdated(_token, _oracleAddress);
66          }
71          function lookupTokenValue(IERC20 _token, uint256 _balance) public view returns (uint256) {
72              AggregatorV3Interface oracle = tokenOracleLookup[_token];
73              if (address(oracle) == address(0x0)) revert OracleNotFound();
74      
75              (, int256 price, , uint256 timestamp, ) = oracle.latestRoundData();
76              if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();
77              if (price <= 0) revert InvalidOraclePrice();
78      
79              // Price is times 10**18 ensure value amount is scaled
80              return (uint256(price) * _balance) / SCALE_FACTOR;
81          }
85          function lookupTokenAmountFromValue(
86              IERC20 _token,
87              uint256 _value
88          ) external view returns (uint256) {
89              AggregatorV3Interface oracle = tokenOracleLookup[_token];
90              if (address(oracle) == address(0x0)) revert OracleNotFound();
91      
92              (, int256 price, , uint256 timestamp, ) = oracle.latestRoundData();
93              if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();
94              if (price <= 0) revert InvalidOraclePrice();
95      
96              // Price is times 10**18 ensure token amount is scaled
97              return (_value * SCALE_FACTOR) / uint256(price);
98          }
103         function lookupTokenValues(
104             IERC20[] memory _tokens,
105             uint256[] memory _balances
106         ) external view returns (uint256) {
107             if (_tokens.length != _balances.length) revert MismatchedArrayLengths();
108     
109             uint256 totalValue = 0;
110             uint256 tokenLength = _tokens.length;
111             for (uint256 i = 0; i < tokenLength; ) {
112                 totalValue += lookupTokenValue(_tokens[i], _balances[i]);
113                 unchecked {
114                     ++i;
115                 }
116             }
117     
118             return totalValue;
119         }
123         function calculateMintAmount(
124             uint256 _currentValueInProtocol,
125             uint256 _newValueAdded,
126             uint256 _existingEzETHSupply
127         ) external pure returns (uint256) {
128             // For first mint, just return the new value added.
129             // Checking both current value and existing supply to guard against gaming the initial mint
130             if (_currentValueInProtocol == 0 || _existingEzETHSupply == 0) {
131                 return _newValueAdded; // value is priced in base units, so divide by scale factor
132             }
133     
134             // Calculate the percentage of value after the deposit
135             uint256 inflationPercentaage = (SCALE_FACTOR * _newValueAdded) /
136                 (_currentValueInProtocol + _newValueAdded);
137     
138             // Calculate the new supply
139             uint256 newEzETHSupply = (_existingEzETHSupply * SCALE_FACTOR) /
140                 (SCALE_FACTOR - inflationPercentaage);
141     
142             // Subtract the old supply from the new supply to get the amount to mint
143             uint256 mintAmount = newEzETHSupply - _existingEzETHSupply;
144     
145             // Sanity check
146             if (mintAmount == 0) revert InvalidTokenAmount();
147     
148             return mintAmount;
149         }
152         function calculateRedeemAmount(
153             uint256 _ezETHBeingBurned,
154             uint256 _existingEzETHSupply,
155             uint256 _currentValueInProtocol
156         ) external pure returns (uint256) {
157             // This is just returning the percentage of TVL that matches the percentage of ezETH being burned
158             uint256 redeemAmount = (_currentValueInProtocol * _ezETHBeingBurned) / _existingEzETHSupply;
159     
160             // Sanity check
161             if (redeemAmount == 0) revert InvalidTokenAmount();
162     
163             return redeemAmount;
164         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


22          function initialize(address roleManagerAdmin) public initializer {
23              if (address(roleManagerAdmin) == address(0x0)) revert InvalidZeroInput();
24      
25              __AccessControl_init();
26      
27              _grantRole(DEFAULT_ADMIN_ROLE, roleManagerAdmin);
28          }
32          function isRoleManagerAdmin(address potentialAddress) external view returns (bool) {
33              return hasRole(DEFAULT_ADMIN_ROLE, potentialAddress);
34          }
38          function isEzETHMinterBurner(address potentialAddress) external view returns (bool) {
39              return hasRole(RX_ETH_MINTER_BURNER, potentialAddress);
40          }
44          function isOperatorDelegatorAdmin(address potentialAddress) external view returns (bool) {
45              return hasRole(OPERATOR_DELEGATOR_ADMIN, potentialAddress);
46          }
50          function isOracleAdmin(address potentialAddress) external view returns (bool) {
51              return hasRole(ORACLE_ADMIN, potentialAddress);
52          }
56          function isRestakeManagerAdmin(address potentialAddress) external view returns (bool) {
57              return hasRole(RESTAKE_MANAGER_ADMIN, potentialAddress);
58          }
62          function isTokenAdmin(address potentialAddress) external view returns (bool) {
63              return hasRole(TOKEN_ADMIN, potentialAddress);
64          }
68          function isNativeEthRestakeAdmin(address potentialAddress) external view returns (bool) {
69              return hasRole(NATIVE_ETH_RESTAKE_ADMIN, potentialAddress);
70          }
74          function isERC20RewardsAdmin(address potentialAddress) external view returns (bool) {
75              return hasRole(ERC20_REWARD_ADMIN, potentialAddress);
76          }
80          function isDepositWithdrawPauser(address potentialAddress) external view returns (bool) {
81              return hasRole(DEPOSIT_WITHDRAW_PAUSER, potentialAddress);
82          }
86          function isBridgeAdmin(address potentialAddress) external view returns (bool) {
87              return hasRole(BRIDGE_ADMIN, potentialAddress);
88          }
92          function isPriceFeedSender(address potentialAddress) external view returns (bool) {
93              return hasRole(PRICE_FEED_SENDER, potentialAddress);
94          }
98          function isWithdrawQueueAdmin(address potentialAddress) external view returns (bool) {
99              return hasRole(WITHDRAW_QUEUE_ADMIN, potentialAddress);
100         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


17          function initialize(
18              IRestakeManager _restakeManager,
19              IERC20Upgradeable _ezETHToken
20          ) public initializer {
21              if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
22              if (address(_ezETHToken) == address(0x0)) revert InvalidZeroInput();
23      
24              restakeManager = _restakeManager;
25              ezETHToken = _ezETHToken;
26          }
29          function getRate() external view returns (uint256) {
30              // Get the total TVL priced in ETH from restakeManager
31              (, , uint256 totalTVL) = restakeManager.calculateTVLs();
32      
33              // Get the total supply of the ezETH token
34              uint256 totalSupply = ezETHToken.totalSupply();
35      
36              // Sanity check
37              if (totalSupply == 0 || totalTVL == 0) revert InvalidZeroInput();
38      
39              // Return the rate
40              return (10 ** 18 * totalTVL) / totalSupply;
41          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


101         function initialize(
102             IRoleManager _roleManager,
103             IEzEthToken _ezETH,
104             IRenzoOracle _renzoOracle,
105             IStrategyManager _strategyManager,
106             IDelegationManager _delegationManager,
107             IDepositQueue _depositQueue
108         ) public initializer {
109             __ReentrancyGuard_init();
110     
111             roleManager = _roleManager;
112             ezETH = _ezETH;
113             renzoOracle = _renzoOracle;
114             strategyManager = _strategyManager;
115             delegationManager = _delegationManager;
116             depositQueue = _depositQueue;
117             paused = false;
118         }
121         function setPaused(bool _paused) external onlyDepositWithdrawPauserAdmin {
122             paused = _paused;
123         }
126         function getOperatorDelegatorsLength() external view returns (uint256) {
127             return operatorDelegators.length;
128         }
131         function addOperatorDelegator(
132             IOperatorDelegator _newOperatorDelegator,
133             uint256 _allocationBasisPoints
134         ) external onlyRestakeManagerAdmin {
135             // Ensure it is not already in the list
136             uint256 odLength = operatorDelegators.length;
137             for (uint256 i = 0; i < odLength; ) {
138                 if (address(operatorDelegators[i]) == address(_newOperatorDelegator))
139                     revert AlreadyAdded();
140                 unchecked {
141                     ++i;
142                 }
143             }
144     
145             // Verify a valid allocation
146             if (_allocationBasisPoints > (100 * BASIS_POINTS)) revert OverMaxBasisPoints();
147     
148             // Add it to the list
149             operatorDelegators.push(_newOperatorDelegator);
150     
151             emit OperatorDelegatorAdded(_newOperatorDelegator);
152     
153             // Set the allocation
154             operatorDelegatorAllocations[_newOperatorDelegator] = _allocationBasisPoints;
155     
156             emit OperatorDelegatorAllocationUpdated(_newOperatorDelegator, _allocationBasisPoints);
157         }
160         function removeOperatorDelegator(
161             IOperatorDelegator _operatorDelegatorToRemove
162         ) external onlyRestakeManagerAdmin {
163             // Remove it from the list
164             uint256 odLength = operatorDelegators.length;
165             for (uint256 i = 0; i < odLength; ) {
166                 if (address(operatorDelegators[i]) == address(_operatorDelegatorToRemove)) {
167                     // Clear the allocation
168                     operatorDelegatorAllocations[_operatorDelegatorToRemove] = 0;
169                     emit OperatorDelegatorAllocationUpdated(_operatorDelegatorToRemove, 0);
170     
171                     // Remove from list
172                     operatorDelegators[i] = operatorDelegators[operatorDelegators.length - 1];
173                     operatorDelegators.pop();
174                     emit OperatorDelegatorRemoved(_operatorDelegatorToRemove);
175                     return;
176                 }
177                 unchecked {
178                     ++i;
179                 }
180             }
181     
182             // If the item was not found, throw an error
183             revert NotFound();
184         }
187         function setOperatorDelegatorAllocation(
188             IOperatorDelegator _operatorDelegator,
189             uint256 _allocationBasisPoints
190         ) external onlyRestakeManagerAdmin {
191             if (address(_operatorDelegator) == address(0x0)) revert InvalidZeroInput();
192             if (_allocationBasisPoints > (100 * BASIS_POINTS)) revert OverMaxBasisPoints();
193     
194             // Ensure the OD is in the list to prevent mis-configuration
195             bool foundOd = false;
196             uint256 odLength = operatorDelegators.length;
197             for (uint256 i = 0; i < odLength; ) {
198                 if (address(operatorDelegators[i]) == address(_operatorDelegator)) {
199                     foundOd = true;
200                     break;
201                 }
202                 unchecked {
203                     ++i;
204                 }
205             }
206             if (!foundOd) revert NotFound();
207     
208             // Set the allocation
209             operatorDelegatorAllocations[_operatorDelegator] = _allocationBasisPoints;
210     
211             emit OperatorDelegatorAllocationUpdated(_operatorDelegator, _allocationBasisPoints);
212         }
215         function setMaxDepositTVL(uint256 _maxDepositTVL) external onlyRestakeManagerAdmin {
216             maxDepositTVL = _maxDepositTVL;
217         }
220         function addCollateralToken(IERC20 _newCollateralToken) external onlyRestakeManagerAdmin {
221             // Ensure it is not already in the list
222             uint256 tokenLength = collateralTokens.length;
223             for (uint256 i = 0; i < tokenLength; ) {
224                 if (address(collateralTokens[i]) == address(_newCollateralToken)) revert AlreadyAdded();
225                 unchecked {
226                     ++i;
227                 }
228             }
229     
230             // Verify the token has 18 decimal precision - pricing calculations will be off otherwise
231             if (IERC20Metadata(address(_newCollateralToken)).decimals() != 18)
232                 revert InvalidTokenDecimals(
233                     18,
234                     IERC20Metadata(address(_newCollateralToken)).decimals()
235                 );
236     
237             // Add it to the list
238             collateralTokens.push(_newCollateralToken);
239     
240             emit CollateralTokenAdded(_newCollateralToken);
241         }
244         function removeCollateralToken(
245             IERC20 _collateralTokenToRemove
246         ) external onlyRestakeManagerAdmin {
247             // Remove it from the list
248             uint256 tokenLength = collateralTokens.length;
249             for (uint256 i = 0; i < tokenLength; ) {
250                 if (address(collateralTokens[i]) == address(_collateralTokenToRemove)) {
251                     collateralTokens[i] = collateralTokens[collateralTokens.length - 1];
252                     collateralTokens.pop();
253                     emit CollateralTokenRemoved(_collateralTokenToRemove);
254                     return;
255                 }
256                 unchecked {
257                     ++i;
258                 }
259             }
260     
261             // If the item was not found, throw an error
262             revert NotFound();
263         }
266         function getCollateralTokensLength() external view returns (uint256) {
267             return collateralTokens.length;
268         }
274         function calculateTVLs() public view returns (uint256[][] memory, uint256[] memory, uint256) {
275             uint256[][] memory operatorDelegatorTokenTVLs = new uint256[][](operatorDelegators.length);
276             uint256[] memory operatorDelegatorTVLs = new uint256[](operatorDelegators.length);
277             uint256 totalTVL = 0;
278     
279             // Iterate through the ODs
280             uint256 odLength = operatorDelegators.length;
281     
282             // flag for withdrawal queue balance set
283             bool withdrawQueueTokenBalanceRecorded = false;
284             address withdrawQueue = address(depositQueue.withdrawQueue());
285     
286             // withdrawalQueue total value
287             uint256 totalWithdrawalQueueValue = 0;
288     
289             for (uint256 i = 0; i < odLength; ) {
290                 // Track the TVL for this OD
291                 uint256 operatorTVL = 0;
292     
293                 // Track the individual token TVLs for this OD - native ETH will be last item in the array
294                 uint256[] memory operatorValues = new uint256[](collateralTokens.length + 1);
295                 operatorDelegatorTokenTVLs[i] = operatorValues;
296     
297                 // Iterate through the tokens and get the value of each
298                 uint256 tokenLength = collateralTokens.length;
299                 for (uint256 j = 0; j < tokenLength; ) {
300                     // Get the value of this token
301     
302                     uint256 operatorBalance = operatorDelegators[i].getTokenBalanceFromStrategy(
303                         collateralTokens[j]
304                     );
305     
306                     // Set the value in the array for this OD
307                     operatorValues[j] = renzoOracle.lookupTokenValue(
308                         collateralTokens[j],
309                         operatorBalance
310                     );
311     
312                     // Add it to the total TVL for this OD
313                     operatorTVL += operatorValues[j];
314     
315                     // record token value of withdraw queue
316                     if (!withdrawQueueTokenBalanceRecorded) {
317                         totalWithdrawalQueueValue += renzoOracle.lookupTokenValue(
318                             collateralTokens[i],
319                             collateralTokens[j].balanceOf(withdrawQueue)
320                         );
321                     }
322     
323                     unchecked {
324                         ++j;
325                     }
326                 }
327     
328                 // Get the value of native ETH staked for the OD
329                 uint256 operatorEthBalance = operatorDelegators[i].getStakedETHBalance();
330     
331                 // Save it to the array for the OD
332                 operatorValues[operatorValues.length - 1] = operatorEthBalance;
333     
334                 // Add it to the total TVL for this OD
335                 operatorTVL += operatorEthBalance;
336     
337                 // Add it to the total TVL for the protocol
338                 totalTVL += operatorTVL;
339     
340                 // Save the TVL for this OD
341                 operatorDelegatorTVLs[i] = operatorTVL;
342     
343                 // Set withdrawQueueTokenBalanceRecorded flag to true
344                 withdrawQueueTokenBalanceRecorded = true;
345     
346                 unchecked {
347                     ++i;
348                 }
349             }
350     
351             // Get the value of native ETH held in the deposit queue and add it to the total TVL
352             totalTVL += address(depositQueue).balance;
353     
354             // Add native ETH help in withdraw Queue and totalWithdrawalQueueValue to totalTVL
355             totalTVL += (address(withdrawQueue).balance + totalWithdrawalQueueValue);
356     
357             return (operatorDelegatorTokenTVLs, operatorDelegatorTVLs, totalTVL);
358         }
362         function chooseOperatorDelegatorForDeposit(
363             uint256[] memory tvls,
364             uint256 totalTVL
365         ) public view returns (IOperatorDelegator) {
366             // Ensure OperatorDelegator list is not empty
367             if (operatorDelegators.length == 0) revert NotFound();
368     
369             // If there is only one operator delegator, return it
370             if (operatorDelegators.length == 1) {
371                 return operatorDelegators[0];
372             }
373     
374             // Otherwise, find the operator delegator with TVL below the threshold
375             uint256 tvlLength = tvls.length;
376             for (uint256 i = 0; i < tvlLength; ) {
377                 if (
378                     tvls[i] <
379                     (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
380                         BASIS_POINTS /
381                         BASIS_POINTS
382                 ) {
383                     return operatorDelegators[i];
384                 }
385     
386                 unchecked {
387                     ++i;
388                 }
389             }
390     
391             // Default to the first operator delegator
392             return operatorDelegators[0];
393         }
400         function chooseOperatorDelegatorForWithdraw(
401             uint256 tokenIndex,
402             uint256 ezETHValue,
403             uint256[][] memory operatorDelegatorTokenTVLs,
404             uint256[] memory operatorDelegatorTVLs,
405             uint256 totalTVL
406         ) public view returns (IOperatorDelegator) {
407             // If there is only one operator delegator, try to use it
408             if (operatorDelegators.length == 1) {
409                 // If the OD doesn't have the tokens, revert
410                 if (operatorDelegatorTokenTVLs[0][tokenIndex] < ezETHValue) {
411                     revert NotFound();
412                 }
413                 return operatorDelegators[0];
414             }
415     
416             // Fnd the operator delegator with TVL above the threshold and with enough tokens
417             uint256 odLength = operatorDelegatorTVLs.length;
418             for (uint256 i = 0; i < odLength; ) {
419                 if (
420                     operatorDelegatorTVLs[i] >
421                     (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
422                         BASIS_POINTS /
423                         BASIS_POINTS &&
424                     operatorDelegatorTokenTVLs[i][tokenIndex] >= ezETHValue
425                 ) {
426                     return operatorDelegators[i];
427                 }
428     
429                 unchecked {
430                     ++i;
431                 }
432             }
433     
434             // If not found, just find one with enough tokens
435             for (uint256 i = 0; i < odLength; ) {
436                 if (operatorDelegatorTokenTVLs[i][tokenIndex] >= ezETHValue) {
437                     return operatorDelegators[i];
438                 }
439     
440                 unchecked {
441                     ++i;
442                 }
443             }
444     
445             // This token cannot be withdrawn
446             revert NotFound();
447         }
451         function getCollateralTokenIndex(IERC20 _collateralToken) public view returns (uint256) {
452             // Find the token index
453             uint256 tokenLength = collateralTokens.length;
454             for (uint256 i = 0; i < tokenLength; ) {
455                 if (collateralTokens[i] == _collateralToken) {
456                     return i;
457                 }
458     
459                 unchecked {
460                     ++i;
461                 }
462             }
463     
464             revert NotFound();
465         }
473         function deposit(IERC20 _collateralToken, uint256 _amount) external {
474             deposit(_collateralToken, _amount, 0);
475         }
491         function deposit(
492             IERC20 _collateralToken,
493             uint256 _amount,
494             uint256 _referralId
495         ) public nonReentrant notPaused {
496             // Verify collateral token is in the list - call will revert if not found
497             uint256 tokenIndex = getCollateralTokenIndex(_collateralToken);
498     
499             // Get the TVLs for each operator delegator and the total TVL
500             (
501                 uint256[][] memory operatorDelegatorTokenTVLs,
502                 uint256[] memory operatorDelegatorTVLs,
503                 uint256 totalTVL
504             ) = calculateTVLs();
505     
506             // Get the value of the collateral token being deposited
507             uint256 collateralTokenValue = renzoOracle.lookupTokenValue(_collateralToken, _amount);
508     
509             // Enforce TVL limit if set, 0 means the check is not enabled
510             if (maxDepositTVL != 0 && totalTVL + collateralTokenValue > maxDepositTVL) {
511                 revert MaxTVLReached();
512             }
513     
514             // Enforce individual token TVL limit if set, 0 means the check is not enabled
515             if (collateralTokenTvlLimits[_collateralToken] != 0) {
516                 // Track the current token's TVL
517                 uint256 currentTokenTVL = 0;
518     
519                 // For each OD, add up the token TVLs
520                 uint256 odLength = operatorDelegatorTokenTVLs.length;
521                 for (uint256 i = 0; i < odLength; ) {
522                     currentTokenTVL += operatorDelegatorTokenTVLs[i][tokenIndex];
523                     unchecked {
524                         ++i;
525                     }
526                 }
527     
528                 // Check if it is over the limit
529                 if (currentTokenTVL + collateralTokenValue > collateralTokenTvlLimits[_collateralToken])
530                     revert MaxTokenTVLReached();
531             }
532     
533             // Determine which operator delegator to use
534             IOperatorDelegator operatorDelegator = chooseOperatorDelegatorForDeposit(
535                 operatorDelegatorTVLs,
536                 totalTVL
537             );
538     
539             // Transfer the collateral token to this address
540             _collateralToken.safeTransferFrom(msg.sender, address(this), _amount);
541     
542             // Check the withdraw buffer and fill if below buffer target
543             uint256 bufferToFill = depositQueue.withdrawQueue().getBufferDeficit(
544                 address(_collateralToken)
545             );
546             if (bufferToFill > 0) {
547                 bufferToFill = (_amount <= bufferToFill) ? _amount : bufferToFill;
548                 // update amount to send to the operator Delegator
549                 _amount -= bufferToFill;
550     
551                 // safe Approve for depositQueue
552                 _collateralToken.safeApprove(address(depositQueue), bufferToFill);
553     
554                 // fill Withdraw Buffer via depositQueue
555                 depositQueue.fillERC20withdrawBuffer(address(_collateralToken), bufferToFill);
556             }
557     
558             // Approve the tokens to the operator delegator
559             _collateralToken.safeApprove(address(operatorDelegator), _amount);
560     
561             // Call deposit on the operator delegator
562             operatorDelegator.deposit(_collateralToken, _amount);
563     
564             // Calculate how much ezETH to mint
565             uint256 ezETHToMint = renzoOracle.calculateMintAmount(
566                 totalTVL,
567                 collateralTokenValue,
568                 ezETH.totalSupply()
569             );
570     
571             // Mint the ezETH
572             ezETH.mint(msg.sender, ezETHToMint);
573     
574             // Emit the deposit event
575             emit Deposit(msg.sender, _collateralToken, _amount, ezETHToMint, _referralId);
576         }
582         function depositETH() external payable {
583             depositETH(0);
584         }
592         function depositETH(uint256 _referralId) public payable nonReentrant notPaused {
593             // Get the total TVL
594             (, , uint256 totalTVL) = calculateTVLs();
595     
596             // Enforce TVL limit if set
597             if (maxDepositTVL != 0 && totalTVL + msg.value > maxDepositTVL) {
598                 revert MaxTVLReached();
599             }
600     
601             // Deposit the remaining ETH into the DepositQueue
602             depositQueue.depositETHFromProtocol{ value: msg.value }();
603     
604             // Calculate how much ezETH to mint
605             uint256 ezETHToMint = renzoOracle.calculateMintAmount(
606                 totalTVL,
607                 msg.value,
608                 ezETH.totalSupply()
609             );
610     
611             // Mint the ezETH
612             ezETH.mint(msg.sender, ezETHToMint);
613     
614             // Emit the deposit event
615             emit Deposit(msg.sender, IERC20(address(0x0)), msg.value, ezETHToMint, _referralId);
616         }
620         function stakeEthInOperatorDelegator(
621             IOperatorDelegator operatorDelegator,
622             bytes calldata pubkey,
623             bytes calldata signature,
624             bytes32 depositDataRoot
625         ) external payable onlyDepositQueue {
626             // Verify the OD is in the list
627             bool found = false;
628             uint256 odLength = operatorDelegators.length;
629             for (uint256 i = 0; i < odLength; ) {
630                 if (operatorDelegators[i] == operatorDelegator) {
631                     found = true;
632                     break;
633                 }
634     
635                 unchecked {
636                     ++i;
637                 }
638             }
639             if (!found) revert NotFound();
640     
641             // Call the OD to stake the ETH
642             operatorDelegator.stakeEth{ value: msg.value }(pubkey, signature, depositDataRoot);
643         }
647         function depositTokenRewardsFromProtocol(
648             IERC20 _token,
649             uint256 _amount
650         ) external onlyDepositQueue {
651             // Get the TVLs for each operator delegator and the total TVL
652             (, uint256[] memory operatorDelegatorTVLs, uint256 totalTVL) = calculateTVLs();
653     
654             // Determine which operator delegator to use
655             IOperatorDelegator operatorDelegator = chooseOperatorDelegatorForDeposit(
656                 operatorDelegatorTVLs,
657                 totalTVL
658             );
659     
660             // Transfer the tokens to this address
661             _token.safeTransferFrom(msg.sender, address(this), _amount);
662     
663             // Approve the tokens to the operator delegator
664             _token.safeApprove(address(operatorDelegator), _amount);
665     
666             // Deposit the tokens into EigenLayer
667             operatorDelegator.deposit(_token, _amount);
668         }
675         function getTotalRewardsEarned() external view returns (uint256) {
676             uint256 totalRewards = 0;
677     
678             // First get the ETH rewards tracked in the deposit queue
679             totalRewards += depositQueue.totalEarned(address(0x0));
680     
681             // For each token, get the total rewards earned from the deposit queue and price it in ETH
682             uint256 tokenLength = collateralTokens.length;
683             for (uint256 i = 0; i < tokenLength; ) {
684                 // Get the amount
685                 uint256 tokenRewardAmount = depositQueue.totalEarned(address(collateralTokens[i]));
686     
687                 // Convert via the price oracle
688                 totalRewards += renzoOracle.lookupTokenValue(collateralTokens[i], tokenRewardAmount);
689     
690                 unchecked {
691                     ++i;
692                 }
693             }
694     
695             // For each OperatorDelegator, get the balance (these are rewards from staking that have not been restaked)
696             // Funds in OD's EigenPod are assumed to be rewards in M1 until exiting validators or withdrawals are supported
697             // Pending unstaked delayed withdrawal amounts are pending being routed into the DepositQueue after a delay
698             uint256 odLength = operatorDelegators.length;
699             for (uint256 i = 0; i < odLength; ) {
700                 totalRewards += address(operatorDelegators[i].eigenPod()).balance;
701                 unchecked {
702                     ++i;
703                 }
704             }
705     
706             return totalRewards;
707         }
709         function setTokenTvlLimit(IERC20 _token, uint256 _limit) external onlyRestakeManagerAdmin {
710             // Verify collateral token is in the list - call will revert if not found
711             getCollateralTokenIndex(_token);
712     
713             // Set the limit
714             collateralTokenTvlLimits[_token] = _limit;
715     
716             emit CollateralTokenTvlUpdated(_token, _limit);
717         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


38          function initialize(IRoleManager _roleManager, address _rewardDestination) public initializer {
39              __ReentrancyGuard_init();
40      
41              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
42              if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();
43      
44              roleManager = _roleManager;
45              rewardDestination = _rewardDestination;
46      
47              emit RewardDestinationUpdated(_rewardDestination);
48          }
52          receive() external payable nonReentrant {
53              _forwardETH();
54          }
58          function forwardRewards() external nonReentrant onlyNativeEthRestakeAdmin {
59              _forwardETH();
60          }
72          function setRewardDestination(
73              address _rewardDestination
74          ) external nonReentrant onlyRestakeManagerAdmin {
75              if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();
76      
77              rewardDestination = _rewardDestination;
78      
79              emit RewardDestinationUpdated(_rewardDestination);
80          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


137         receive() external payable {}
142         function supportsInterface(
143             bytes4 interfaceId
144         ) public view virtual override(IERC165, AccessControl) returns (bool) {
145             return
146                 interfaceId == type(IERC1155Receiver).interfaceId ||
147                 super.supportsInterface(interfaceId);
148         }
154         function isOperation(bytes32 id) public view virtual returns (bool) {
155             return getTimestamp(id) > 0;
156         }
161         function isOperationPending(bytes32 id) public view virtual returns (bool) {
162             return getTimestamp(id) > _DONE_TIMESTAMP;
163         }
168         function isOperationReady(bytes32 id) public view virtual returns (bool) {
169             uint256 timestamp = getTimestamp(id);
170             return timestamp > _DONE_TIMESTAMP && timestamp <= block.timestamp;
171         }
176         function isOperationDone(bytes32 id) public view virtual returns (bool) {
177             return getTimestamp(id) == _DONE_TIMESTAMP;
178         }
184         function getTimestamp(bytes32 id) public view virtual returns (uint256) {
185             return _timestamps[id];
186         }
193         function getMinDelay() public view virtual returns (uint256) {
194             return _minDelay;
195         }
201         function hashOperation(
202             address target,
203             uint256 value,
204             bytes calldata data,
205             bytes32 predecessor,
206             bytes32 salt
207         ) public pure virtual returns (bytes32) {
208             return keccak256(abi.encode(target, value, data, predecessor, salt));
209         }
215         function hashOperationBatch(
216             address[] calldata targets,
217             uint256[] calldata values,
218             bytes[] calldata payloads,
219             bytes32 predecessor,
220             bytes32 salt
221         ) public pure virtual returns (bytes32) {
222             return keccak256(abi.encode(targets, values, payloads, predecessor, salt));
223         }
234         function schedule(
235             address target,
236             uint256 value,
237             bytes calldata data,
238             bytes32 predecessor,
239             bytes32 salt,
240             uint256 delay
241         ) public virtual onlyRole(PROPOSER_ROLE) {
242             bytes32 id = hashOperation(target, value, data, predecessor, salt);
243             _schedule(id, delay);
244             emit CallScheduled(id, 0, target, value, data, predecessor, delay);
245             if (salt != bytes32(0)) {
246                 emit CallSalt(id, salt);
247             }
248         }
259         function scheduleBatch(
260             address[] calldata targets,
261             uint256[] calldata values,
262             bytes[] calldata payloads,
263             bytes32 predecessor,
264             bytes32 salt,
265             uint256 delay
266         ) public virtual onlyRole(PROPOSER_ROLE) {
267             require(targets.length == values.length, "TimelockController: length mismatch");
268             require(targets.length == payloads.length, "TimelockController: length mismatch");
269     
270             bytes32 id = hashOperationBatch(targets, values, payloads, predecessor, salt);
271             _schedule(id, delay);
272             for (uint256 i = 0; i < targets.length; ++i) {
273                 emit CallScheduled(id, i, targets[i], values[i], payloads[i], predecessor, delay);
274             }
275             if (salt != bytes32(0)) {
276                 emit CallSalt(id, salt);
277             }
278         }
296         function cancel(bytes32 id) public virtual onlyRole(CANCELLER_ROLE) {
297             require(isOperationPending(id), "TimelockController: operation cannot be cancelled");
298             delete _timestamps[id];
299     
300             emit Cancelled(id);
301         }
315         function execute(
316             address target,
317             uint256 value,
318             bytes calldata payload,
319             bytes32 predecessor,
320             bytes32 salt
321         ) public payable virtual onlyRoleOrOpenRole(EXECUTOR_ROLE) {
322             bytes32 id = hashOperation(target, value, payload, predecessor, salt);
323     
324             _beforeCall(id, predecessor);
325             _execute(target, value, payload);
326             emit CallExecuted(id, 0, target, value, payload);
327             _afterCall(id);
328         }
342         function executeBatch(
343             address[] calldata targets,
344             uint256[] calldata values,
345             bytes[] calldata payloads,
346             bytes32 predecessor,
347             bytes32 salt
348         ) public payable virtual onlyRoleOrOpenRole(EXECUTOR_ROLE) {
349             require(targets.length == values.length, "TimelockController: length mismatch");
350             require(targets.length == payloads.length, "TimelockController: length mismatch");
351     
352             bytes32 id = hashOperationBatch(targets, values, payloads, predecessor, salt);
353     
354             _beforeCall(id, predecessor);
355             for (uint256 i = 0; i < targets.length; ++i) {
356                 address target = targets[i];
357                 uint256 value = values[i];
358                 bytes calldata payload = payloads[i];
359                 _execute(target, value, payload);
360                 emit CallExecuted(id, i, target, value, payload);
361             }
362             _afterCall(id);
363         }
402         function updateDelay(uint256 newDelay) external virtual {
403             require(msg.sender == address(this), "TimelockController: caller must be timelock");
404             emit MinDelayChange(_minDelay, newDelay);
405             _minDelay = newDelay;
406         }
411         function onERC721Received(
412             address,
413             address,
414             uint256,
415             bytes memory
416         ) public virtual override returns (bytes4) {
417             return this.onERC721Received.selector;
418         }
423         function onERC1155Received(
424             address,
425             address,
426             uint256,
427             uint256,
428             bytes memory
429         ) public virtual override returns (bytes4) {
430             return this.onERC1155Received.selector;
431         }
436         function onERC1155BatchReceived(
437             address,
438             address,
439             uint256[] memory,
440             uint256[] memory,
441             bytes memory
442         ) public virtual override returns (bytes4) {
443             return this.onERC1155BatchReceived.selector;
444         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


64          function initialize(
65              IRoleManager _roleManager,
66              IRestakeManager _restakeManager,
67              IEzEthToken _ezETH,
68              IRenzoOracle _renzoOracle,
69              uint256 _coolDownPeriod,
70              TokenWithdrawBuffer[] calldata _withdrawalBufferTarget
71          ) external initializer {
72              if (
73                  address(_roleManager) == address(0) ||
74                  address(_ezETH) == address(0) ||
75                  address(_renzoOracle) == address(0) ||
76                  address(_restakeManager) == address(0) ||
77                  _withdrawalBufferTarget.length == 0 ||
78                  _coolDownPeriod == 0
79              ) revert InvalidZeroInput();
80      
81              __Pausable_init();
82      
83              roleManager = _roleManager;
84              restakeManager = _restakeManager;
85              ezETH = _ezETH;
86              renzoOracle = _renzoOracle;
87              coolDownPeriod = _coolDownPeriod;
88              for (uint256 i = 0; i < _withdrawalBufferTarget.length; ) {
89                  if (
90                      _withdrawalBufferTarget[i].asset == address(0) ||
91                      _withdrawalBufferTarget[i].bufferAmount == 0
92                  ) revert InvalidZeroInput();
93                  withdrawalBufferTarget[_withdrawalBufferTarget[i].asset] = _withdrawalBufferTarget[i]
94                      .bufferAmount;
95                  unchecked {
96                      ++i;
97                  }
98              }
99          }
106         function updateWithdrawBufferTarget(
107             TokenWithdrawBuffer[] calldata _newBufferTarget
108         ) external onlyWithdrawQueueAdmin {
109             if (_newBufferTarget.length == 0) revert InvalidZeroInput();
110             for (uint256 i = 0; i < _newBufferTarget.length; ) {
111                 if (_newBufferTarget[i].asset == address(0) || _newBufferTarget[i].bufferAmount == 0)
112                     revert InvalidZeroInput();
113                 emit WithdrawBufferTargetUpdated(
114                     withdrawalBufferTarget[_newBufferTarget[i].asset],
115                     _newBufferTarget[i].bufferAmount
116                 );
117                 withdrawalBufferTarget[_newBufferTarget[i].asset] = _newBufferTarget[i].bufferAmount;
118                 unchecked {
119                     ++i;
120                 }
121             }
122         }
129         function updateCoolDownPeriod(uint256 _newCoolDownPeriod) external onlyWithdrawQueueAdmin {
130             if (_newCoolDownPeriod == 0) revert InvalidZeroInput();
131             emit CoolDownPeriodUpdated(coolDownPeriod, _newCoolDownPeriod);
132             coolDownPeriod = _newCoolDownPeriod;
133         }
139         function pause() external onlyWithdrawQueueAdmin {
140             _pause();
141         }
147         function unpause() external onlyWithdrawQueueAdmin {
148             _unpause();
149         }
156         function getAvailableToWithdraw(address _asset) public view returns (uint256) {
157             if (_asset != IS_NATIVE) {
158                 return IERC20(_asset).balanceOf(address(this)) - claimReserve[_asset];
159             } else {
160                 return address(this).balance - claimReserve[_asset];
161             }
162         }
170         function getBufferDeficit(address _asset) public view returns (uint256) {
171             uint256 availableToWithdraw = getAvailableToWithdraw(_asset);
172             return
173                 withdrawalBufferTarget[_asset] > availableToWithdraw
174                     ? withdrawalBufferTarget[_asset] - availableToWithdraw
175                     : 0;
176         }
182         function fillEthWithdrawBuffer() external payable nonReentrant onlyDepositQueue {
183             emit EthBufferFilled(msg.value);
184         }
192         function fillERC20WithdrawBuffer(
193             address _asset,
194             uint256 _amount
195         ) external nonReentrant onlyDepositQueue {
196             if (_asset == address(0) || _amount == 0) revert InvalidZeroInput();
197             IERC20(_asset).safeTransferFrom(msg.sender, address(this), _amount);
198             emit ERC20BufferFilled(_asset, _amount);
199         }
206         function withdraw(uint256 _amount, address _assetOut) external nonReentrant {
207             // check for 0 values
208             if (_amount == 0 || _assetOut == address(0)) revert InvalidZeroInput();
209     
210             // check if provided assetOut is supported
211             if (withdrawalBufferTarget[_assetOut] == 0) revert UnsupportedWithdrawAsset();
212     
213             // transfer ezETH tokens to this address
214             IERC20(address(ezETH)).safeTransferFrom(msg.sender, address(this), _amount);
215     
216             // calculate totalTVL
217             (, , uint256 totalTVL) = restakeManager.calculateTVLs();
218     
219             // Calculate amount to Redeem in ETH
220             uint256 amountToRedeem = renzoOracle.calculateRedeemAmount(
221                 _amount,
222                 ezETH.totalSupply(),
223                 totalTVL
224             );
225     
226             // update amount in claim asset, if claim asset is not ETH
227             if (_assetOut != IS_NATIVE) {
228                 // Get ERC20 asset equivalent amount
229                 amountToRedeem = renzoOracle.lookupTokenAmountFromValue(
230                     IERC20(_assetOut),
231                     amountToRedeem
232                 );
233             }
234     
235             // revert if amount to redeem is greater than withdrawBufferTarget
236             if (amountToRedeem > getAvailableToWithdraw(_assetOut)) revert NotEnoughWithdrawBuffer();
237     
238             // increment the withdrawRequestNonce
239             withdrawRequestNonce++;
240     
241             // add withdraw request for msg.sender
242             withdrawRequests[msg.sender].push(
243                 WithdrawRequest(
244                     _assetOut,
245                     withdrawRequestNonce,
246                     amountToRedeem,
247                     _amount,
248                     block.timestamp
249                 )
250             );
251     
252             // add redeem amount to claimReserve of claim asset
253             claimReserve[_assetOut] += amountToRedeem;
254     
255             emit WithdrawRequestCreated(
256                 withdrawRequestNonce,
257                 msg.sender,
258                 _assetOut,
259                 amountToRedeem,
260                 _amount,
261                 withdrawRequests[msg.sender].length - 1
262             );
263         }
270         function getOutstandingWithdrawRequests(address user) public view returns (uint256) {
271             return withdrawRequests[user].length;
272         }
279         function claim(uint256 withdrawRequestIndex) external nonReentrant {
280             // check if provided withdrawRequest Index is valid
281             if (withdrawRequestIndex >= withdrawRequests[msg.sender].length)
282                 revert InvalidWithdrawIndex();
283     
284             WithdrawRequest memory _withdrawRequest = withdrawRequests[msg.sender][
285                 withdrawRequestIndex
286             ];
287             if (block.timestamp - _withdrawRequest.createdAt < coolDownPeriod) revert EarlyClaim();
288     
289             // subtract value from claim reserve for claim asset
290             claimReserve[_withdrawRequest.collateralToken] -= _withdrawRequest.amountToRedeem;
291     
292             // delete the withdraw request
293             withdrawRequests[msg.sender][withdrawRequestIndex] = withdrawRequests[msg.sender][
294                 withdrawRequests[msg.sender].length - 1
295             ];
296             withdrawRequests[msg.sender].pop();
297     
298             // burn ezETH locked for withdraw request
299             ezETH.burn(address(this), _withdrawRequest.ezETHLocked);
300     
301             // send selected redeem asset to user
302             if (_withdrawRequest.collateralToken == IS_NATIVE) {
303                 payable(msg.sender).transfer(_withdrawRequest.amountToRedeem);
304             } else {
305                 IERC20(_withdrawRequest.collateralToken).transfer(
306                     msg.sender,
307                     _withdrawRequest.amountToRedeem
308                 );
309             }
310             // emit the event
311             emit WithdrawRequestClaimed(_withdrawRequest);
312         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


33          function initialize(IRoleManager _roleManager) public initializer {
34              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
35      
36              __ERC20_init("ezETH", "Renzo Restaked ETH");
37              roleManager = _roleManager;
38          }
41          function mint(address to, uint256 amount) external onlyMinterBurner {
42              _mint(to, amount);
43          }
46          function burn(address from, uint256 amount) external onlyMinterBurner {
47              _burn(from, amount);
48          }
51          function setPaused(bool _paused) external onlyTokenAdmin {
52              paused = _paused;
53          }
77          function name() public view virtual override returns (string memory) {
78              return "Renzo Restaked ETH";
79          }
85          function symbol() public view virtual override returns (string memory) {
86              return "ezETH";
87          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## NC044 - Consider adding formal verification proofs:

Consider using formal verification to mathematically prove that your code does what is intended, and does not have any edge cases with unexpected behavior. The solidity compiler itself has this functionality [built in based off of SMTChecker](https://docs.soliditylang.org/en/latest/smtchecker.html#smtchecker-and-formal-verification).


```solidity
File: Various Files


None

```


## NC045 - Common functions should be refactored to a common base contract:

The functions below have the same implementation as is seen in other files. The functions should be refactored into functions of a common base contract.


<details>
<summary>Click to show 9 findings</summary>

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


/// @audit seen in contest/contracts/Bridge/L1/xRenzoBridge.sol
    receive() external payable {}

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L542:542

```solidity
File: contracts/TimelockController.sol


/// @audit seen in contest/contracts/Bridge/L1/xRenzoBridge.sol
    receive() external payable {}

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L137:137

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


/// @audit seen in contest/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol
    function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {
        if (_newXRenzoBridgeL1 == address(0)) revert InvalidZeroInput();
        emit XRenzoBridgeL1Updated(_newXRenzoBridgeL1, xRenzoBridgeL1);
        xRenzoBridgeL1 = _newXRenzoBridgeL1;
    }

/// @audit seen in contest/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol
    function unPause() external onlyOwner {
        _unpause();
    }

/// @audit seen in contest/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol
    function pause() external onlyOwner {
        _pause();
    }

/// @audit seen in contest/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol
    function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {
        if (_newXRenzoDeposit == address(0)) revert InvalidZeroInput();
        emit XRenzoDepositUpdated(_newXRenzoDeposit, address(xRenzoDeposit));
        xRenzoDeposit = IxRenzoDeposit(_newXRenzoDeposit);
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L130:134

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


/// @audit seen in contest/contracts/Oracle/Binance/WBETHShim.sol
    function decimals() external pure returns (uint8) {
        return 18;
    }

/// @audit seen in contest/contracts/Oracle/Binance/WBETHShim.sol
    function version() external pure returns (uint256) {
        return 1;
    }

/// @audit seen in contest/contracts/Oracle/Binance/WBETHShim.sol
    function getRoundData(uint80) external pure returns (uint80, int256, uint256, uint256, uint80) {
        revert NotImplemented();
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L42:44

</details>

## NC046 - Polymorphic functions make security audits more time-consuming and error-prone:

The instances below point to one of two functions with the same name. Consider naming each function differently, in order to make code navigation and analysis easier.


```solidity
File: contracts/RestakeManager.sol


491         function deposit(
592         function depositETH(uint256 _referralId) public payable nonReentrant notPaused {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

## NC047 - Event names should use CamelCase:

According to the Solidity [style guide](https://docs.soliditylang.org/en/latest/style-guide.html#event-names) event names should be in `CamelCase`.


```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


25          event EzETHMinted(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


25          event CCIPEthChainSelectorUpdated(uint64 newSourceChainSelector, uint64 oldSourceChainSelector);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


23          event ETHStakedFromQueue(
41          event FullWithdrawalETHReceived(uint256 amount);
21          event ETHDepositedFromProtocol(uint256 amount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

## NC048 - Consider using `AccessControlDefaultAdminRules` rather than `AccessControl`:

`AccessControlDefaultAdminRules` implements multiple [security best practices](https://docs.openzeppelin.com/contracts/4.x/api/access#AccessControlDefaultAdminRules) on top of the normal `AccessControl` rules, so consider using it instead.


```solidity
File: contracts/TimelockController.sol


25      contract TimelockController is AccessControl, IERC721Receiver, IERC1155Receiver {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

## NC049 - Missing timelock for critical parameter change:

Timelocks prevent users from being surprised by changes.


```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


137             xRenzoDeposit = IxRenzoDeposit(_newXRenzoDeposit);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


133             xRenzoDeposit = IxRenzoDeposit(_newXRenzoDeposit);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


123             lockbox = _lockbox;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

## NC050 - Setters should prevent re-setting of the same value:

This especially problematic when the setter also emits the same value, which may be confusing to offline parsers.


<details>
<summary>Click to show 11 findings</summary>

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


36          function setOracleAddress(AggregatorV3Interface _oracleAddress) external onlyOwner {
37              if (address(_oracleAddress) == address(0)) revert InvalidZeroInput();
38              // Verify that the pricing of the oracle is less than or equal to 18 decimals - pricing calculations will be off otherwise
39              if (_oracleAddress.decimals() > 18)
40                  revert InvalidTokenDecimals(18, _oracleAddress.decimals());
41      
42              emit OracleAddressUpdated(address(_oracleAddress), address(oracle));
43              oracle = _oracleAddress;
44          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


134         function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {
135             if (_newXRenzoDeposit == address(0)) revert InvalidZeroInput();
136             emit XRenzoDepositUpdated(_newXRenzoDeposit, address(xRenzoDeposit));
137             xRenzoDeposit = IxRenzoDeposit(_newXRenzoDeposit);
138         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


130         function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {
131             if (_newXRenzoDeposit == address(0)) revert InvalidZeroInput();
132             emit XRenzoDepositUpdated(_newXRenzoDeposit, address(xRenzoDeposit));
133             xRenzoDeposit = IxRenzoDeposit(_newXRenzoDeposit);
134         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


466         function setAllowedBridgeSweeper(address _sweeper, bool _allowed) external onlyOwner {
467             allowedBridgeSweepers[_sweeper] = _allowed;
468     
469             emit BridgeSweeperAddressUpdated(_sweeper, _allowed);
470         }
501         function setOraclePriceFeed(IRenzoOracleL2 _oracle) external onlyOwner {
502             emit OraclePriceFeedUpdated(address(_oracle), address(oracle));
503             oracle = _oracle;
504         }
511         function setReceiverPriceFeed(address _receiver) external onlyOwner {
512             emit ReceiverPriceFeedUpdated(_receiver, receiver);
513             receiver = _receiver;
514         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


121         function setLockbox(address _lockbox) public {
122             if (msg.sender != FACTORY) revert IXERC20_NotFactory();
123             lockbox = _lockbox;
124     
125             emit LockboxSet(_lockbox);
126         }
135         function setLimits(
136             address _bridge,
137             uint256 _mintingLimit,
138             uint256 _burningLimit
139         ) external onlyOwner {
140             _changeMinterLimit(_bridge, _mintingLimit);
141             _changeBurnerLimit(_bridge, _burningLimit);
142             emit BridgeLimitsSet(_mintingLimit, _burningLimit, _bridge);
143         }
230         function _changeMinterLimit(address _bridge, uint256 _limit) internal {
231             uint256 _oldLimit = bridges[_bridge].minterParams.maxLimit;
232             uint256 _currentLimit = mintingCurrentLimitOf(_bridge);
233             bridges[_bridge].minterParams.maxLimit = _limit;
234     
235             bridges[_bridge].minterParams.currentLimit = _calculateNewCurrentLimit(
236                 _limit,
237                 _oldLimit,
238                 _currentLimit
239             );
240     
241             bridges[_bridge].minterParams.ratePerSecond = _limit / _DURATION;
242             bridges[_bridge].minterParams.timestamp = block.timestamp;
243         }
252         function _changeBurnerLimit(address _bridge, uint256 _limit) internal {
253             uint256 _oldLimit = bridges[_bridge].burnerParams.maxLimit;
254             uint256 _currentLimit = burningCurrentLimitOf(_bridge);
255             bridges[_bridge].burnerParams.maxLimit = _limit;
256     
257             bridges[_bridge].burnerParams.currentLimit = _calculateNewCurrentLimit(
258                 _limit,
259                 _oldLimit,
260                 _currentLimit
261             );
262     
263             bridges[_bridge].burnerParams.ratePerSecond = _limit / _DURATION;
264             bridges[_bridge].burnerParams.timestamp = block.timestamp;
265         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


106         function setTokenStrategy(
107             IERC20 _token,
108             IStrategy _strategy
109         ) external nonReentrant onlyOperatorDelegatorAdmin {
110             if (address(_token) == address(0x0)) revert InvalidZeroInput();
111     
112             tokenStrategyMapping[_token] = _strategy;
113             emit TokenStrategyUpdated(_token, _strategy);
114         }
117         function setDelegateAddress(
118             address _delegateAddress,
119             ISignatureUtils.SignatureWithExpiry memory approverSignatureAndExpiry,
120             bytes32 approverSalt
121         ) external nonReentrant onlyOperatorDelegatorAdmin {
122             if (address(_delegateAddress) == address(0x0)) revert InvalidZeroInput();
123             if (address(delegateAddress) != address(0x0)) revert DelegateAddressAlreadySet();
124     
125             delegateAddress = _delegateAddress;
126     
127             delegationManager.delegateTo(delegateAddress, approverSignatureAndExpiry, approverSalt);
128     
129             emit DelegationAddressUpdated(_delegateAddress);
130         }
132         function setBaseGasAmountSpent(
133             uint256 _baseGasAmountSpent
134         ) external nonReentrant onlyOperatorDelegatorAdmin {
135             if (_baseGasAmountSpent == 0) revert InvalidZeroInput();
136             emit BaseGasAmountSpentUpdated(baseGasAmountSpent, _baseGasAmountSpent);
137             baseGasAmountSpent = _baseGasAmountSpent;
138         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


87          function setWithdrawQueue(IWithdrawQueue _withdrawQueue) external onlyRestakeManagerAdmin {
88              if (address(_withdrawQueue) == address(0)) revert InvalidZeroInput();
89              emit WithdrawQueueUpdated(address(withdrawQueue), address(_withdrawQueue));
90              withdrawQueue = _withdrawQueue;
91          }
93          function setFeeConfig(
94              address _feeAddress,
95              uint256 _feeBasisPoints
96          ) external onlyRestakeManagerAdmin {
97              // Verify address is set if basis points are non-zero
98              if (_feeBasisPoints > 0) {
99                  if (_feeAddress == address(0x0)) revert InvalidZeroInput();
100             }
101     
102             // Verify basis points are not over 100%
103             if (_feeBasisPoints > 10000) revert OverMaxBasisPoints();
104     
105             feeAddress = _feeAddress;
106             feeBasisPoints = _feeBasisPoints;
107     
108             emit FeeConfigUpdated(_feeAddress, _feeBasisPoints);
109         }
112         function setRestakeManager(IRestakeManager _restakeManager) external onlyRestakeManagerAdmin {
113             if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
114     
115             restakeManager = _restakeManager;
116     
117             emit RestakeManagerUpdated(_restakeManager);
118         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


54          function setOracleAddress(
55              IERC20 _token,
56              AggregatorV3Interface _oracleAddress
57          ) external nonReentrant onlyOracleAdmin {
58              if (address(_token) == address(0x0)) revert InvalidZeroInput();
59      
60              // Verify that the pricing of the oracle is 18 decimals - pricing calculations will be off otherwise
61              if (_oracleAddress.decimals() != 18)
62                  revert InvalidTokenDecimals(18, _oracleAddress.decimals());
63      
64              tokenOracleLookup[_token] = _oracleAddress;
65              emit OracleAddressUpdated(_token, _oracleAddress);
66          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


121         function setPaused(bool _paused) external onlyDepositWithdrawPauserAdmin {
122             paused = _paused;
123         }
187         function setOperatorDelegatorAllocation(
188             IOperatorDelegator _operatorDelegator,
189             uint256 _allocationBasisPoints
190         ) external onlyRestakeManagerAdmin {
191             if (address(_operatorDelegator) == address(0x0)) revert InvalidZeroInput();
192             if (_allocationBasisPoints > (100 * BASIS_POINTS)) revert OverMaxBasisPoints();
193     
194             // Ensure the OD is in the list to prevent mis-configuration
195             bool foundOd = false;
196             uint256 odLength = operatorDelegators.length;
197             for (uint256 i = 0; i < odLength; ) {
198                 if (address(operatorDelegators[i]) == address(_operatorDelegator)) {
199                     foundOd = true;
200                     break;
201                 }
202                 unchecked {
203                     ++i;
204                 }
205             }
206             if (!foundOd) revert NotFound();
207     
208             // Set the allocation
209             operatorDelegatorAllocations[_operatorDelegator] = _allocationBasisPoints;
210     
211             emit OperatorDelegatorAllocationUpdated(_operatorDelegator, _allocationBasisPoints);
212         }
215         function setMaxDepositTVL(uint256 _maxDepositTVL) external onlyRestakeManagerAdmin {
216             maxDepositTVL = _maxDepositTVL;
217         }
709         function setTokenTvlLimit(IERC20 _token, uint256 _limit) external onlyRestakeManagerAdmin {
710             // Verify collateral token is in the list - call will revert if not found
711             getCollateralTokenIndex(_token);
712     
713             // Set the limit
714             collateralTokenTvlLimits[_token] = _limit;
715     
716             emit CollateralTokenTvlUpdated(_token, _limit);
717         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


72          function setRewardDestination(
73              address _rewardDestination
74          ) external nonReentrant onlyRestakeManagerAdmin {
75              if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();
76      
77              rewardDestination = _rewardDestination;
78      
79              emit RewardDestinationUpdated(_rewardDestination);
80          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


51          function setPaused(bool _paused) external onlyTokenAdmin {
52              paused = _paused;
53          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## NC051 - High cyclomatic complexity:

Consider breaking down these blocks into more manageable units, by splitting things into utility functions, by reducing nesting, and by using early returns.


```solidity
File: contracts/Delegation/OperatorDelegator.sol


265         function completeQueuedWithdrawal(
266             IDelegationManager.Withdrawal calldata withdrawal,
267             IERC20[] calldata tokens,
268             uint256 middlewareTimesIndex
269         ) external nonReentrant onlyNativeEthRestakeAdmin {
270             uint256 gasBefore = gasleft();
271             if (tokens.length != withdrawal.strategies.length) revert MismatchedArrayLengths();
272     
273             // complete the queued withdrawal from EigenLayer with receiveAsToken set to true
274             delegationManager.completeQueuedWithdrawal(withdrawal, tokens, middlewareTimesIndex, true);
275     
276             IWithdrawQueue withdrawQueue = restakeManager.depositQueue().withdrawQueue();
277             for (uint256 i; i < tokens.length; ) {
278                 if (address(tokens[i]) == address(0)) revert InvalidZeroInput();
279     
280                 // deduct queued shares for tracking TVL
281                 queuedShares[address(tokens[i])] -= withdrawal.shares[i];
282     
283                 // check if token is not Native ETH
284                 if (address(tokens[i]) != IS_NATIVE) {
285                     // Check the withdraw buffer and fill if below buffer target
286                     uint256 bufferToFill = withdrawQueue.getBufferDeficit(address(tokens[i]));
287     
288                     // get balance of this contract
289                     uint256 balanceOfToken = tokens[i].balanceOf(address(this));
290                     if (bufferToFill > 0) {
291                         bufferToFill = (balanceOfToken <= bufferToFill) ? balanceOfToken : bufferToFill;
292     
293                         // update amount to send to the operator Delegator
294                         balanceOfToken -= bufferToFill;
295     
296                         // safe Approve for depositQueue
297                         tokens[i].safeApprove(address(restakeManager.depositQueue()), bufferToFill);
298     
299                         // fill Withdraw Buffer via depositQueue
300                         restakeManager.depositQueue().fillERC20withdrawBuffer(
301                             address(tokens[i]),
302                             bufferToFill
303                         );
304                     }
305     
306                     // Deposit remaining tokens back to eigenLayer
307                     if (balanceOfToken > 0) {
308                         _deposit(tokens[i], balanceOfToken);
309                     }
310                 }
311                 unchecked {
312                     ++i;
313                 }
314             }
315     
316             // emits the Withdraw Completed event with withdrawalRoot
317             emit WithdrawCompleted(
318                 delegationManager.calculateWithdrawalRoot(withdrawal),
319                 withdrawal.strategies,
320                 withdrawal.shares
321             );
322             // record current spent gas
323             _recordGas(gasBefore);
324         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

## NC052 - Use of override is unnecessary:

Starting with Solidity version 0.8.8, using the override keyword when the function solely overrides an interface function, and the function doesn't exist in multiple base contracts, is unnecessary.


<details>
<summary>Click to show 15 findings</summary>

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


    function _ccipReceive(
        Client.Any2EVMMessage memory any2EvmMessage
    ) internal override whenNotPaused {
        address _ccipSender = abi.decode(any2EvmMessage.sender, (address));
        uint64 _ccipSourceChainSelector = any2EvmMessage.sourceChainSelector;
        // Verify origin on the price feed
        if (_ccipSender != xRenzoBridgeL1) revert InvalidSender(xRenzoBridgeL1, _ccipSender);
        // Verify Source chain of the message
        if (_ccipSourceChainSelector != ccipEthChainSelector)
            revert InvalidSourceChain(ccipEthChainSelector, _ccipSourceChainSelector);
        (uint256 _price, uint256 _timestamp) = abi.decode(any2EvmMessage.data, (uint256, uint256));
        xRenzoDeposit.updatePrice(_price, _timestamp);
        emit MessageReceived(
            any2EvmMessage.messageId,
            _ccipSourceChainSelector,
            _ccipSender,
            _price,
            _timestamp
        );
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L66:85

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


    function updatePrice(uint256 _price, uint256 _timestamp) external override {
        if (msg.sender != receiver) revert InvalidSender(receiver, msg.sender);
        _updatePrice(_price, _timestamp);
    }

    function getRate() external view override returns (uint256) {
        return lastPrice;
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L456:458

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


    function supportsInterface(
        bytes4 interfaceId
    ) public view override(ERC165Upgradeable) returns (bool) {
        return
            interfaceId == type(IOptimismMintableERC20).interfaceId ||
            super.supportsInterface(interfaceId);
    }

    function remoteToken() public view override returns (address) {
        return l1Token;
    }

    function bridge() public view override returns (address) {
        return optimismBridge;
    }

    function mint(address _to, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
        XERC20.mint(_to, _amount);
    }

    function burn(address _from, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
        XERC20.burn(_from, _amount);
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L68:70

```solidity
File: contracts/TimelockController.sol


    function supportsInterface(
        bytes4 interfaceId
    ) public view virtual override(IERC165, AccessControl) returns (bool) {
        return
            interfaceId == type(IERC1155Receiver).interfaceId ||
            super.supportsInterface(interfaceId);
    }

    function onERC721Received(
        address,
        address,
        uint256,
        bytes memory
    ) public virtual override returns (bytes4) {
        return this.onERC721Received.selector;
    }

    function onERC1155Received(
        address,
        address,
        uint256,
        uint256,
        bytes memory
    ) public virtual override returns (bytes4) {
        return this.onERC1155Received.selector;
    }

    function onERC1155BatchReceived(
        address,
        address,
        uint256[] memory,
        uint256[] memory,
        bytes memory
    ) public virtual override returns (bytes4) {
        return this.onERC1155BatchReceived.selector;
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L436:444

```solidity
File: contracts/token/EzEthToken.sol


    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
        super._beforeTokenTransfer(from, to, amount);

        // If not paused return success
        if (!paused) {
            return;
        }

        // If paused, only minting and burning should be allowed
        if (from != address(0) && to != address(0)) {
            revert ContractPaused();
        }
    }

    function name() public view virtual override returns (string memory) {
        return "Renzo Restaked ETH";
    }

    function symbol() public view virtual override returns (string memory) {
        return "ezETH";
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L85:87

</details>

## NC053 - Unused import:

The identifier is imported but never used within the file


```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


6       import { IXERC20 } from "../../xERC20/interfaces/IXERC20.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


4       import { Client } from "contest/@chainlink/contracts-ccip/src/v0.8/ccip/libraries/Client.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


6       import { XERC20Lockbox } from "../XERC20Lockbox.sol";


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

## NC054 - Put all system-wide constants in one file:

Putting all the system-wide constants in a single file improves code readability, makes it easier to understand the basic configuration and limitations of the system, and makes maintenance easier.


<details>
<summary>Click to show 22 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


61          uint8 public constant EXPECTED_DECIMALS = 18;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L61:61

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


13          uint256 public constant MAX_TIME_WINDOW = 86400 + 60; // 24 hours + 60 seconds


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L13:13

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


37          uint8 public constant EXPECTED_DECIMALS = 18;


40          uint32 public constant FEE_BASIS = 10000;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L40:40

```solidity
File: contracts/Delegation/OperatorDelegator.sol


26          address public constant IS_NATIVE = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L26:26

```solidity
File: contracts/Deposits/DepositQueue.sol


13          address public constant IS_NATIVE = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L13:13

```solidity
File: contracts/Permissions/RoleManagerStorage.sol


11          bytes32 public constant RX_ETH_MINTER_BURNER = keccak256("RX_ETH_MINTER_BURNER");


14          bytes32 public constant OPERATOR_DELEGATOR_ADMIN = keccak256("OPERATOR_DELEGATOR_ADMIN");


17          bytes32 public constant ORACLE_ADMIN = keccak256("ORACLE_ADMIN");


20          bytes32 public constant RESTAKE_MANAGER_ADMIN = keccak256("RESTAKE_MANAGER_ADMIN");


23          bytes32 public constant TOKEN_ADMIN = keccak256("TOKEN_ADMIN");


26          bytes32 public constant NATIVE_ETH_RESTAKE_ADMIN = keccak256("NATIVE_ETH_RESTAKE_ADMIN");


29          bytes32 public constant ERC20_REWARD_ADMIN = keccak256("ERC20_REWARD_ADMIN");


32          bytes32 public constant DEPOSIT_WITHDRAW_PAUSER = keccak256("DEPOSIT_WITHDRAW_PAUSER");


39          bytes32 public constant BRIDGE_ADMIN = keccak256("BRIDGE_ADMIN");


42          bytes32 public constant PRICE_FEED_SENDER = keccak256("PRICE_FEED_SENDER");


47          bytes32 public constant WITHDRAW_QUEUE_ADMIN = keccak256("WITHDRAW_QUEUE_ADMIN");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManagerStorage.sol#L47:47

```solidity
File: contracts/TimelockController.sol


26          bytes32 public constant TIMELOCK_ADMIN_ROLE = keccak256("TIMELOCK_ADMIN_ROLE");


27          bytes32 public constant PROPOSER_ROLE = keccak256("PROPOSER_ROLE");


28          bytes32 public constant EXECUTOR_ROLE = keccak256("EXECUTOR_ROLE");


29          bytes32 public constant CANCELLER_ROLE = keccak256("CANCELLER_ROLE");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L29:29

```solidity
File: contracts/Withdraw/WithdrawQueueStorage.sol


10          address public constant IS_NATIVE = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueueStorage.sol#L10:10

</details>

## NC055 - Add inline comments for unnamed variables:

`function foo(address x, address)` -> `function foo(address x, address /* y */)`


<details>
<summary>Click to show 4 findings</summary>

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


69          function xReceive(
70              bytes32 _transferId,
71              uint256,
72              address,
73              address _originSender,
74              uint32 _origin,
75              bytes memory _callData
76          ) external onlySource(_originSender, _origin) whenNotPaused returns (bytes memory) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


42          function getRoundData(uint80) external pure returns (uint80, int256, uint256, uint256, uint80) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


42          function getRoundData(uint80) external pure returns (uint80, int256, uint256, uint256, uint80) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/TimelockController.sol


411         function onERC721Received(
412             address,
413             address,
414             uint256,
415             bytes memory
416         ) public virtual override returns (bytes4) {
423         function onERC1155Received(
424             address,
425             address,
426             uint256,
427             uint256,
428             bytes memory
429         ) public virtual override returns (bytes4) {
436         function onERC1155BatchReceived(
437             address,
438             address,
439             uint256[] memory,
440             uint256[] memory,
441             bytes memory
442         ) public virtual override returns (bytes4) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

</details>

## NC056 - Consider adding emergency-stop functionality:

Adding a way to quickly halt protocol functionality in an emergency, rather than having to pause individual contracts one-by-one, will make in-progress hack mitigation faster and much less stressful.


<details>
<summary>Click to show 5 findings</summary>

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


11      contract RenzoOracleL2 is Initializable, OwnableUpgradeable, RenzoOracleL2StorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


14      contract Receiver is CCIPReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


10      contract ConnextReceiver is IXReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


27      contract xRenzoDeposit is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


16      contract XERC20 is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

</details>

## NC057 - Named imports of parent contracts are missing:

  


<details>
<summary>Click to show 28 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


17          IXReceiver,
20          xRenzoBridgeStorageV1
18          Initializable,
19          ReentrancyGuardUpgradeable,


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridgeStorage.sol


18      abstract contract xRenzoBridgeStorageV1 is IxRenzoBridge {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridgeStorage.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


11      contract RenzoOracleL2 is Initializable, OwnableUpgradeable, RenzoOracleL2StorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol


7       abstract contract RenzoOracleL2StorageV1 is IRenzoOracleL2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


28          Initializable,
32          xRenzoDepositStorageV3
30          ReentrancyGuardUpgradeable,
31          IRateProvider,


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


52      abstract contract xRenzoDepositStorageV3 is xRenzoDepositStorageV2 {
47      abstract contract xRenzoDepositStorageV2 is xRenzoDepositStorageV1 {
9       abstract contract xRenzoDepositStorageV1 is IxRenzoDeposit {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


17          Initializable,


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


14      contract XERC20Factory is Initializable, IXERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


11      contract XERC20Lockbox is Initializable, IXERC20Lockbox {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


14      contract OptimismMintableXERC20Factory is Initializable, XERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


19          ReentrancyGuardUpgradeable,
20          OperatorDelegatorStorageV4
18          Initializable,


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegatorStorage.sol


59      abstract contract OperatorDelegatorStorageV4 is OperatorDelegatorStorageV3 {
51      abstract contract OperatorDelegatorStorageV3 is OperatorDelegatorStorageV2 {
16      abstract contract OperatorDelegatorStorageV1 is IOperatorDelegator {
46      abstract contract OperatorDelegatorStorageV2 is OperatorDelegatorStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegatorStorage.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


10      contract DepositQueue is Initializable, ReentrancyGuardUpgradeable, DepositQueueStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueueStorage.sol


26      abstract contract DepositQueueStorageV2 is DepositQueueStorageV1 {
9       abstract contract DepositQueueStorageV1 is IDepositQueue {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueueStorage.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


15      contract WBETHShim is Initializable, WBETHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShimStorage.sol


7       abstract contract WBETHShimStorageV1 is AggregatorV3Interface {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


15      contract METHShim is Initializable, METHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShimStorage.sol


7       abstract contract METHShimStorageV1 is AggregatorV3Interface {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


17          RenzoOracleStorageV1
14          IRenzoOracle,
16          ReentrancyGuardUpgradeable,
15          Initializable,


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


14      contract RoleManager is IRoleManager, AccessControlUpgradeable, RoleManagerStorageV3 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/Permissions/RoleManagerStorage.sol


37      contract RoleManagerStorageV2 is RoleManagerStorageV1 {
45      contract RoleManagerStorageV3 is RoleManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManagerStorage.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


9       contract BalancerRateProvider is Initializable, IRateProvider, BalancerRateProviderStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


26      contract RestakeManager is Initializable, ReentrancyGuardUpgradeable, RestakeManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/RestakeManagerStorage.sol


64      abstract contract RestakeManagerStorageV2 is RestakeManagerStorageV1 {
15      abstract contract RestakeManagerStorageV1 is IRestakeManager {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


16      contract RewardHandler is Initializable, ReentrancyGuardUpgradeable, RewardHandlerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


25      contract TimelockController is AccessControl, IERC721Receiver, IERC1155Receiver {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


12          Initializable,
14          ReentrancyGuardUpgradeable,
15          WithdrawQueueStorageV1
13          PausableUpgradeable,


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


13      contract EzEthToken is Initializable, ERC20Upgradeable, IEzEthToken, EzEthTokenStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## NC059 - Unspecific Compiler Version Pragma:

  


<details>
<summary>Click to show 5 findings</summary>

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


pragma solidity >=0.8.4 <0.9.0;

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L2:2

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


pragma solidity >=0.8.4 <0.9.0;

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L2:2

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


pragma solidity >=0.8.4 <0.9.0;

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L2:2

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


pragma solidity >=0.8.4 <0.9.0;

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L2:2

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


pragma solidity >=0.8.4 <0.9.0;

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L2:2

</details>

## NC060 - Consider using `SafeTransferLib.safeTransferETH()` or `Address.sendValue()` for clearer semantic meaning:

These Functions indicate their purpose with their name more clearly than using low-level calls.


<details>
<summary>Click to show 5 findings</summary>

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


403             (bool success, ) = payable(msg.sender).call{ value: feeCollected }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


131                 (bool _success, ) = payable(_to).call{ value: _amount }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


520                 (bool success, ) = destination.call{ value: remainingAmount }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


286             (bool success, ) = payable(msg.sender).call{ value: gasRefund }("");
168                 (bool success, ) = feeAddress.call{ value: feeAmount }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


68              (bool success, ) = rewardDestination.call{ value: balance }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

</details>

## NC061 - Style guide: State and local variables should be named using lowerCamelCase:

The Solidity style guide says to use mixedCase for local and state variable names. Note that while OpenZeppelin may not follow this advice, it still is the recommended way of naming variables.


```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


31          address public FACTORY;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


18          IXERC20 public XERC20;
23          IERC20 public ERC20;
29          bool public IS_NATIVE;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

## NC062 - Unnecessary cast:

The variable is being cast to its own type


<details>
<summary>Click to show 4 findings</summary>

```solidity
File: contracts/Delegation/OperatorDelegator.sol


122             if (address(_delegateAddress) == address(0x0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


23              if (address(roleManagerAdmin) == address(0x0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


355             totalTVL += (address(withdrawQueue).balance + totalWithdrawalQueueValue);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


42              if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();
75              if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

</details>

## NC063 - Unusual loop variable:

The normal name for loop variables is i, and when there is a nested loop, to use j. Not following this convention may lead to some reviewer confusion.


```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


167             for (uint256 _i; _i < _bridgesLength; ++_i) {
168                 XERC20(_xerc20).setLimits(_bridges[_i], _minterLimits[_i], _burnerLimits[_i]);
169             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


109             for (uint256 _i; _i < _bridgesLength; ++_i) {
110                 OptimismMintableXERC20(_xerc20).setLimits(
111                     _bridges[_i],
112                     _minterLimits[_i],
113                     _burnerLimits[_i]
114                 );
115             }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

## NC064 - Use the latest solidity (prior to 0.8.20 if on L2s) for deployment:

Since deployed contracts should not use floating pragmas, I've flagged all instances where a version prior to 0.8.19 is allowed by the version pragma


<details>
<summary>Click to show 10 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


2       pragma solidity ^0.8.13;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/Connext/libraries/LibConnextStorage.sol


2       pragma solidity ^0.8.0;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/libraries/LibConnextStorage.sol#L0:0

```solidity
File: contracts/Bridge/Connext/libraries/TokenId.sol


2       pragma solidity ^0.8.0;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/libraries/TokenId.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


2       pragma solidity >=0.8.4 <0.9.0;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


2       pragma solidity >=0.8.4 <0.9.0;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


2       pragma solidity >=0.8.4 <0.9.0;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


2       pragma solidity >=0.8.4 <0.9.0;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


2       pragma solidity >=0.8.4 <0.9.0;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/TimelockController.sol


4       pragma solidity ^0.8.0;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


2       pragma solidity ^0.8.9;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## NC065 - Visibility should be set explicitly rather than defaulting to internal:

Visibility of state variables should be set explicitly rather than defaulting to internal.


```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


31          address immutable blastStandardBridge;
32          address immutable registry;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


20          string constant INVALID_0_INPUT = "Invalid 0 input";
23          uint256 constant SCALE_FACTOR = 10 ** 18;
26          uint256 constant MAX_TIME_WINDOW = 86400 + 60; // 24 hours + 60 seconds


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


38          uint256 constant BASIS_POINTS = 100;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

## NC067 - Event declarations should have NatSpec @param annotations:

Documents a parameter just like in Doxygen (must be followed by parameter name)


<details>
<summary>Click to show 12 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


25          event EzETHMinted(
34          event MessageSent(
43          event ConnextMessageSent(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


15          event OracleAddressUpdated(address newOracle, address oldOracle);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


24          event XRenzoBridgeL1Updated(address newBridgeAddress, address oldBridgeAddress);
25          event CCIPEthChainSelectorUpdated(uint64 newSourceChainSelector, uint64 oldSourceChainSelector);
26          event XRenzoDepositUpdated(address newRenzoDeposit, address oldRenzoDeposit);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


23          event XRenzoBridgeL1Updated(address newBridgeAddress, address oldBridgeAddress);
24          event ConnextEthChainDomainUpdated(uint32 newSourceChainDomain, uint32 oldSourceChainDomain);
25          event XRenzoDepositUpdated(address newRenzoDeposit, address oldRenzoDeposit);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


42          event PriceUpdated(uint256 price, uint256 timestamp);
43          event Deposit(address indexed user, uint256 amountIn, uint256 amountOut);
44          event BridgeSweeperAddressUpdated(address sweeper, bool allowed);
45          event BridgeSwept(
51          event OraclePriceFeedUpdated(address newOracle, address oldOracle);
52          event ReceiverPriceFeedUpdated(address newReceiver, address oldReceiver);
53          event SweeperBridgeFeeCollected(address sweeper, uint256 feeCollected);
54          event BridgeFeeShareUpdated(uint256 oldBridgeFeeShare, uint256 newBridgeFeeShare);
55          event SweepBatchSizeUpdated(uint256 oldSweepBatchSize, uint256 newSweepBatchSize);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


28          event TokenStrategyUpdated(IERC20 token, IStrategy strategy);
29          event DelegationAddressUpdated(address delegateAddress);
30          event RewardsForwarded(address rewardDestination, uint256 amount);
32          event WithdrawStarted(
43          event WithdrawCompleted(bytes32 withdrawalRoot, IStrategy[] strategies, uint256[] shares);
45          event GasSpent(address admin, uint256 gasSpent);
46          event GasRefunded(address admin, uint256 gasRefunded);
47          event BaseGasAmountSpentUpdated(uint256 oldBaseGasAmountSpent, uint256 newBaseGasAmountSpent);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


15          event RewardsDeposited(IERC20 token, uint256 amount);
17          event FeeConfigUpdated(address feeAddress, uint256 feeBasisPoints);
19          event RestakeManagerUpdated(IRestakeManager restakeManager);
21          event ETHDepositedFromProtocol(uint256 amount);
23          event ETHStakedFromQueue(
30          event ProtocolFeesPaid(IERC20 token, uint256 amount, address destination);
32          event GasRefunded(address admin, uint256 gasRefunded);
35          event WithdrawQueueUpdated(address oldWithdrawQueue, address newWithdrawQueue);
38          event BufferFilled(address token, uint256 amount);
41          event FullWithdrawalETHReceived(uint256 amount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


35          event OracleAddressUpdated(IERC20 token, AggregatorV3Interface oracleAddress);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


30          event OperatorDelegatorAdded(IOperatorDelegator od);
31          event OperatorDelegatorRemoved(IOperatorDelegator od);
32          event OperatorDelegatorAllocationUpdated(IOperatorDelegator od, uint256 allocation);
34          event CollateralTokenAdded(IERC20 token);
35          event CollateralTokenRemoved(IERC20 token);
41          event Deposit(
50          event UserWithdrawStarted(
59          event UserWithdrawCompleted(
68          event CollateralTokenTvlUpdated(IERC20 token, uint256 tvl);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


29          event RewardDestinationUpdated(address rewardDestination);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


38          event CallScheduled(
51          event CallExecuted(
62          event CallSalt(bytes32 indexed id, bytes32 salt);
67          event Cancelled(bytes32 indexed id);
72          event MinDelayChange(uint256 oldDuration, uint256 newDuration);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


19          event WithdrawBufferTargetUpdated(uint256 oldBufferTarget, uint256 newBufferTarget);
21          event CoolDownPeriodUpdated(uint256 oldCoolDownPeriod, uint256 newCoolDownPeriod);
23          event EthBufferFilled(uint256 amount);
25          event ERC20BufferFilled(address asset, uint256 amount);
27          event WithdrawRequestCreated(
36          event WithdrawRequestClaimed(WithdrawRequest withdrawRequest);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## NC068 - Event declarations should have NatSpec @dev annotations:

Explain to a developer any extra details


<details>
<summary>Click to show 10 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


43          event ConnextMessageSent(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


15          event OracleAddressUpdated(address newOracle, address oldOracle);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


24          event XRenzoBridgeL1Updated(address newBridgeAddress, address oldBridgeAddress);
25          event CCIPEthChainSelectorUpdated(uint64 newSourceChainSelector, uint64 oldSourceChainSelector);
26          event XRenzoDepositUpdated(address newRenzoDeposit, address oldRenzoDeposit);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


23          event XRenzoBridgeL1Updated(address newBridgeAddress, address oldBridgeAddress);
24          event ConnextEthChainDomainUpdated(uint32 newSourceChainDomain, uint32 oldSourceChainDomain);
25          event XRenzoDepositUpdated(address newRenzoDeposit, address oldRenzoDeposit);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


42          event PriceUpdated(uint256 price, uint256 timestamp);
43          event Deposit(address indexed user, uint256 amountIn, uint256 amountOut);
44          event BridgeSweeperAddressUpdated(address sweeper, bool allowed);
45          event BridgeSwept(
51          event OraclePriceFeedUpdated(address newOracle, address oldOracle);
52          event ReceiverPriceFeedUpdated(address newReceiver, address oldReceiver);
53          event SweeperBridgeFeeCollected(address sweeper, uint256 feeCollected);
54          event BridgeFeeShareUpdated(uint256 oldBridgeFeeShare, uint256 newBridgeFeeShare);
55          event SweepBatchSizeUpdated(uint256 oldSweepBatchSize, uint256 newSweepBatchSize);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


28          event TokenStrategyUpdated(IERC20 token, IStrategy strategy);
29          event DelegationAddressUpdated(address delegateAddress);
30          event RewardsForwarded(address rewardDestination, uint256 amount);
32          event WithdrawStarted(
43          event WithdrawCompleted(bytes32 withdrawalRoot, IStrategy[] strategies, uint256[] shares);
45          event GasSpent(address admin, uint256 gasSpent);
46          event GasRefunded(address admin, uint256 gasRefunded);
47          event BaseGasAmountSpentUpdated(uint256 oldBaseGasAmountSpent, uint256 newBaseGasAmountSpent);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


15          event RewardsDeposited(IERC20 token, uint256 amount);
17          event FeeConfigUpdated(address feeAddress, uint256 feeBasisPoints);
19          event RestakeManagerUpdated(IRestakeManager restakeManager);
21          event ETHDepositedFromProtocol(uint256 amount);
23          event ETHStakedFromQueue(
30          event ProtocolFeesPaid(IERC20 token, uint256 amount, address destination);
32          event GasRefunded(address admin, uint256 gasRefunded);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


30          event OperatorDelegatorAdded(IOperatorDelegator od);
31          event OperatorDelegatorRemoved(IOperatorDelegator od);
32          event OperatorDelegatorAllocationUpdated(IOperatorDelegator od, uint256 allocation);
34          event CollateralTokenAdded(IERC20 token);
35          event CollateralTokenRemoved(IERC20 token);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


29          event RewardDestinationUpdated(address rewardDestination);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


19          event WithdrawBufferTargetUpdated(uint256 oldBufferTarget, uint256 newBufferTarget);
21          event CoolDownPeriodUpdated(uint256 oldCoolDownPeriod, uint256 newCoolDownPeriod);
23          event EthBufferFilled(uint256 amount);
25          event ERC20BufferFilled(address asset, uint256 amount);
27          event WithdrawRequestCreated(
36          event WithdrawRequestClaimed(WithdrawRequest withdrawRequest);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## NC069 - Function definitions should have NatSpec @dev annotations:

Explain to a developer any extra details


<details>
<summary>Click to show 20 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


10          function getXERC20(address erc20) external view returns (address xerc20);
12          function getERC20(address xerc20) external view returns (address erc20);
14          function getLockbox(address erc20) external view returns (address xerc20);
18          function bridgeERC20To(
39          constructor(address _blastStandardBridge, address _registry) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


23          function initialize(AggregatorV3Interface _oracle) public initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


43          constructor(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


52          constructor(address _connext, address _xRenzoBridgeL1, uint32 _connextEthChainDomain) {
69          function xReceive(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


277         function getBridgeFeeShare(uint256 _amountIn) public view returns (uint256) {
289         function getMintRate() public view returns (uint256, uint256) {
396         function _recoverBridgeFee() internal {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


61          function initialize(
76          function __XERC20_init(
96          function mint(address _user, uint256 _amount) public virtual {
107         function burn(address _user, uint256 _amount) public virtual {
121         function setLockbox(address _lockbox) public {
152         function mintingMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
163         function burningMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
174         function mintingCurrentLimitOf(address _bridge) public view returns (uint256 _limit) {
190         function burningCurrentLimitOf(address _bridge) public view returns (uint256 _limit) {
205         function _useMinterLimits(address _bridge, uint256 _change) internal {
217         function _useBurnerLimits(address _bridge, uint256 _change) internal {
230         function _changeMinterLimit(address _bridge, uint256 _limit) internal {
252         function _changeBurnerLimit(address _bridge, uint256 _limit) internal {
276         function _calculateNewCurrentLimit(
302         function _getCurrentLimit(
328         function _burnWithCaller(address _caller, address _user, uint256 _amount) internal {
348         function _mintWithCaller(address _caller, address _user, uint256 _amount) internal {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


54          function initialize(
74          function deployXERC20(
105         function deployLockbox(
135         function _deployXERC20(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


44          function initialize(address _xerc20, address _erc20, bool _isNative) public initializer {
54          function depositNative() public payable {
66          function deposit(uint256 _amount) external {
79          function depositTo(address _to, uint256 _amount) external {
91          function depositNativeTo(address _to) public payable {
103         function withdraw(uint256 _amount) external {
114         function withdrawTo(address _to, uint256 _amount) external {
125         function _withdraw(address _to, uint256 _amount) internal {
145         function _deposit(address _to, uint256 _amount) internal {
157         receive() external payable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


35          function initialize(
48          function supportsInterface(
56          function remoteToken() public view override returns (address) {
60          function bridge() public view override returns (address) {
64          function mint(address _to, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
68          function burn(address _from, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


37          function deployOptimismMintableXERC20(
73          function _deployOptimismMintableXERC20(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


132         function setBaseGasAmountSpent(
162         function _deposit(IERC20 _token, uint256 _tokenAmount) internal returns (uint256 shares) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


283         function _refundGas(uint256 initialGas) internal {
294         function _checkAndFillETHWithdrawBuffer(uint256 _amount) internal {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/IRestakeManager.sol


8           function stakeEthInOperatorDelegator(
14          function depositTokenRewardsFromProtocol(IERC20 _token, uint256 _amount) external;
15          function depositQueue() external view returns (IDepositQueue);
17          function calculateTVLs() external view returns (uint256[][] memory, uint256[] memory, uint256);
19          function depositETH() external payable;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/IRestakeManager.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


29          function decimals() external pure returns (uint8) {
33          function description() external pure returns (string memory) {
37          function version() external pure returns (uint256) {
46          function latestRoundData()


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


29          function decimals() external pure returns (uint8) {
33          function description() external pure returns (string memory) {
37          function version() external pure returns (uint256) {
46          function latestRoundData()


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


152         function calculateRedeemAmount(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


709         function setTokenTvlLimit(IERC20 _token, uint256 _limit) external onlyRestakeManagerAdmin {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


62          function _forwardETH() internal {
72          function setRewardDestination(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


315         function execute(
342         function executeBatch(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


64          function initialize(
156         function getAvailableToWithdraw(address _asset) public view returns (uint256) {
206         function withdraw(uint256 _amount, address _assetOut) external nonReentrant {
270         function getOutstandingWithdrawRequests(address user) public view returns (uint256) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## NC070 - Function definitions should have NatSpec @notice annotations:

Explain to an end user what this does


<details>
<summary>Click to show 24 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


10          function getXERC20(address erc20) external view returns (address xerc20);
12          function getERC20(address xerc20) external view returns (address erc20);
14          function getLockbox(address erc20) external view returns (address xerc20);
18          function bridgeERC20To(
39          constructor(address _blastStandardBridge, address _registry) {
56          function bridgeTo(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


65          constructor() {
70          function initialize(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


19          constructor() {
23          function initialize(AggregatorV3Interface _oracle) public initializer {
36          function setOracleAddress(AggregatorV3Interface _oracleAddress) external onlyOwner {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


43          constructor(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


52          constructor(address _connext, address _xRenzoBridgeL1, uint32 _connextEthChainDomain) {
69          function xReceive(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


59          constructor() {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


50          constructor() {
96          function mint(address _user, uint256 _amount) public virtual {
107         function burn(address _user, uint256 _amount) public virtual {
121         function setLockbox(address _lockbox) public {
152         function mintingMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
163         function burningMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
174         function mintingCurrentLimitOf(address _bridge) public view returns (uint256 _limit) {
190         function burningCurrentLimitOf(address _bridge) public view returns (uint256 _limit) {
205         function _useMinterLimits(address _bridge, uint256 _change) internal {
217         function _useBurnerLimits(address _bridge, uint256 _change) internal {
230         function _changeMinterLimit(address _bridge, uint256 _limit) internal {
252         function _changeBurnerLimit(address _bridge, uint256 _limit) internal {
276         function _calculateNewCurrentLimit(
302         function _getCurrentLimit(
328         function _burnWithCaller(address _caller, address _user, uint256 _amount) internal {
348         function _mintWithCaller(address _caller, address _user, uint256 _amount) internal {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


44          constructor() {
74          function deployXERC20(
105         function deployLockbox(
135         function _deployXERC20(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


33          constructor() {
54          function depositNative() public payable {
66          function deposit(uint256 _amount) external {
79          function depositTo(address _to, uint256 _amount) external {
91          function depositNativeTo(address _to) public payable {
103         function withdraw(uint256 _amount) external {
114         function withdrawTo(address _to, uint256 _amount) external {
125         function _withdraw(address _to, uint256 _amount) internal {
145         function _deposit(address _to, uint256 _amount) internal {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


24          constructor() {
48          function supportsInterface(
56          function remoteToken() public view override returns (address) {
60          function bridge() public view override returns (address) {
64          function mint(address _to, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
68          function burn(address _from, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


19          constructor() {
37          function deployOptimismMintableXERC20(
73          function _deployOptimismMintableXERC20(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


69          constructor() {
74          function initialize(
101         function activateRestaking() external nonReentrant onlyNativeEthRestakeAdmin {
106         function setTokenStrategy(
117         function setDelegateAddress(
132         function setBaseGasAmountSpent(
143         function deposit(
172         function getStrategyIndex(IStrategy _strategy) public view returns (uint256) {
327         function getTokenBalanceFromStrategy(IERC20 token) external view returns (uint256) {
338         function getStakedETHBalance() external view returns (uint256) {
349         function stakeEth(
364         function verifyWithdrawalCredentials(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


69          constructor() {
74          function initialize(IRoleManager _roleManager) public initializer {
93          function setFeeConfig(
112         function setRestakeManager(IRestakeManager _restakeManager) external onlyRestakeManagerAdmin {
123         function depositETHFromProtocol() external payable onlyRestakeManager {
163         receive() external payable nonReentrant {
187         function stakeEthFromQueue(
211         function stakeEthFromQueueMulti(
254         function sweepERC20(IERC20 token) external onlyERC20RewardsAdmin {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/IRestakeManager.sol


8           function stakeEthInOperatorDelegator(
14          function depositTokenRewardsFromProtocol(IERC20 _token, uint256 _amount) external;
15          function depositQueue() external view returns (IDepositQueue);
17          function calculateTVLs() external view returns (uint256[][] memory, uint256[] memory, uint256);
19          function depositETH() external payable;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/IRestakeManager.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


18          constructor() {
23          function initialize(IStakedTokenV2 _wBETHToken) public initializer {
29          function decimals() external pure returns (uint8) {
33          function description() external pure returns (string memory) {
37          function version() external pure returns (uint256) {
42          function getRoundData(uint80) external pure returns (uint80, int256, uint256, uint256, uint80) {
46          function latestRoundData()


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


18          constructor() {
23          function initialize(IMethStaking _methStaking) public initializer {
29          function decimals() external pure returns (uint8) {
33          function description() external pure returns (string memory) {
37          function version() external pure returns (uint256) {
42          function getRoundData(uint80) external pure returns (uint80, int256, uint256, uint256, uint80) {
46          function latestRoundData()


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


39          constructor() {
44          function initialize(IRoleManager _roleManager) public initializer {
54          function setOracleAddress(
71          function lookupTokenValue(IERC20 _token, uint256 _balance) public view returns (uint256) {
85          function lookupTokenAmountFromValue(
103         function lookupTokenValues(
123         function calculateMintAmount(
152         function calculateRedeemAmount(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


17          constructor() {
22          function initialize(address roleManagerAdmin) public initializer {
32          function isRoleManagerAdmin(address potentialAddress) external view returns (bool) {
38          function isEzETHMinterBurner(address potentialAddress) external view returns (bool) {
44          function isOperatorDelegatorAdmin(address potentialAddress) external view returns (bool) {
50          function isOracleAdmin(address potentialAddress) external view returns (bool) {
56          function isRestakeManagerAdmin(address potentialAddress) external view returns (bool) {
62          function isTokenAdmin(address potentialAddress) external view returns (bool) {
68          function isNativeEthRestakeAdmin(address potentialAddress) external view returns (bool) {
74          function isERC20RewardsAdmin(address potentialAddress) external view returns (bool) {
80          function isDepositWithdrawPauser(address potentialAddress) external view returns (bool) {
86          function isBridgeAdmin(address potentialAddress) external view returns (bool) {
92          function isPriceFeedSender(address potentialAddress) external view returns (bool) {
98          function isWithdrawQueueAdmin(address potentialAddress) external view returns (bool) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


12          constructor() {
17          function initialize(
29          function getRate() external view returns (uint256) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


96          constructor() {
101         function initialize(
121         function setPaused(bool _paused) external onlyDepositWithdrawPauserAdmin {
126         function getOperatorDelegatorsLength() external view returns (uint256) {
131         function addOperatorDelegator(
160         function removeOperatorDelegator(
187         function setOperatorDelegatorAllocation(
215         function setMaxDepositTVL(uint256 _maxDepositTVL) external onlyRestakeManagerAdmin {
220         function addCollateralToken(IERC20 _newCollateralToken) external onlyRestakeManagerAdmin {
244         function removeCollateralToken(
266         function getCollateralTokensLength() external view returns (uint256) {
274         function calculateTVLs() public view returns (uint256[][] memory, uint256[] memory, uint256) {
362         function chooseOperatorDelegatorForDeposit(
400         function chooseOperatorDelegatorForWithdraw(
451         function getCollateralTokenIndex(IERC20 _collateralToken) public view returns (uint256) {
620         function stakeEthInOperatorDelegator(
647         function depositTokenRewardsFromProtocol(
709         function setTokenTvlLimit(IERC20 _token, uint256 _limit) external onlyRestakeManagerAdmin {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


33          constructor() {
38          function initialize(IRoleManager _roleManager, address _rewardDestination) public initializer {
52          receive() external payable nonReentrant {
58          function forwardRewards() external nonReentrant onlyNativeEthRestakeAdmin {
62          function _forwardETH() internal {
72          function setRewardDestination(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


87          constructor(
137         receive() external payable {}
142         function supportsInterface(
154         function isOperation(bytes32 id) public view virtual returns (bool) {
161         function isOperationPending(bytes32 id) public view virtual returns (bool) {
168         function isOperationReady(bytes32 id) public view virtual returns (bool) {
176         function isOperationDone(bytes32 id) public view virtual returns (bool) {
184         function getTimestamp(bytes32 id) public view virtual returns (uint256) {
193         function getMinDelay() public view virtual returns (uint256) {
201         function hashOperation(
215         function hashOperationBatch(
234         function schedule(
259         function scheduleBatch(
283         function _schedule(bytes32 id, uint256 delay) private {
296         function cancel(bytes32 id) public virtual onlyRole(CANCELLER_ROLE) {
315         function execute(
342         function executeBatch(
368         function _execute(address target, uint256 value, bytes calldata data) internal virtual {
376         function _beforeCall(bytes32 id, bytes32 predecessor) private view {
387         function _afterCall(bytes32 id) private {
402         function updateDelay(uint256 newDelay) external virtual {
411         function onERC721Received(
423         function onERC1155Received(
436         function onERC1155BatchReceived(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


57          constructor() {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


28          constructor() {
33          function initialize(IRoleManager _roleManager) public initializer {
41          function mint(address to, uint256 amount) external onlyMinterBurner {
46          function burn(address from, uint256 amount) external onlyMinterBurner {
51          function setPaused(bool _paused) external onlyTokenAdmin {
56          function _beforeTokenTransfer(
77          function name() public view virtual override returns (string memory) {
85          function symbol() public view virtual override returns (string memory) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## NC071 - Interface declarations should have NatSpec @title annotations:

A title that should describe the contract/interface


```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


9       interface IXERC20Registry {
17      interface L1StandardBridge {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/IRestakeManager.sol


7       interface IRestakeManager {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/IRestakeManager.sol#L0:0

## NC072 - Interface declarations should have NatSpec @author annotations:

The name of the author


```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


9       interface IXERC20Registry {
17      interface L1StandardBridge {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/IRestakeManager.sol


7       interface IRestakeManager {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/IRestakeManager.sol#L0:0

## NC073 - Interface declarations should have NatSpec @notice annotations:

Explain to an end user what this does


```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


9       interface IXERC20Registry {
17      interface L1StandardBridge {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/IRestakeManager.sol


7       interface IRestakeManager {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/IRestakeManager.sol#L0:0

## NC074 - Interface declarations should have NatSpec @dev annotations:

Explain to a developer any extra details


```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


9       interface IXERC20Registry {
17      interface L1StandardBridge {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/IRestakeManager.sol


7       interface IRestakeManager {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/IRestakeManager.sol#L0:0

## NC075 - Abstract contract declarations should have NatSpec @title annotations:

A title that should describe the contract/interface


<details>
<summary>Click to show 12 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridgeStorage.sol


18      abstract contract xRenzoBridgeStorageV1 is IxRenzoBridge {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridgeStorage.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol


7       abstract contract RenzoOracleL2StorageV1 is IRenzoOracleL2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


9       abstract contract xRenzoDepositStorageV1 is IxRenzoDeposit {
47      abstract contract xRenzoDepositStorageV2 is xRenzoDepositStorageV1 {
52      abstract contract xRenzoDepositStorageV3 is xRenzoDepositStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegatorStorage.sol


46      abstract contract OperatorDelegatorStorageV2 is OperatorDelegatorStorageV1 {
51      abstract contract OperatorDelegatorStorageV3 is OperatorDelegatorStorageV2 {
59      abstract contract OperatorDelegatorStorageV4 is OperatorDelegatorStorageV3 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegatorStorage.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueueStorage.sol


9       abstract contract DepositQueueStorageV1 is IDepositQueue {
26      abstract contract DepositQueueStorageV2 is DepositQueueStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueueStorage.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShimStorage.sol


7       abstract contract WBETHShimStorageV1 is AggregatorV3Interface {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShimStorage.sol


7       abstract contract METHShimStorageV1 is AggregatorV3Interface {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracleStorage.sol


8       abstract contract RenzoOracleStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracleStorage.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProviderStorage.sol


7       abstract contract BalancerRateProviderStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProviderStorage.sol#L0:0

```solidity
File: contracts/RestakeManagerStorage.sol


15      abstract contract RestakeManagerStorageV1 is IRestakeManager {
64      abstract contract RestakeManagerStorageV2 is RestakeManagerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandlerStorage.sol


6       abstract contract RewardHandlerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandlerStorage.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueueStorage.sol


9       abstract contract WithdrawQueueStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueueStorage.sol#L0:0

</details>

## NC076 - Abstract contract declarations should have NatSpec @author annotations:

The name of the author


<details>
<summary>Click to show 12 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridgeStorage.sol


18      abstract contract xRenzoBridgeStorageV1 is IxRenzoBridge {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridgeStorage.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol


7       abstract contract RenzoOracleL2StorageV1 is IRenzoOracleL2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


9       abstract contract xRenzoDepositStorageV1 is IxRenzoDeposit {
47      abstract contract xRenzoDepositStorageV2 is xRenzoDepositStorageV1 {
52      abstract contract xRenzoDepositStorageV3 is xRenzoDepositStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegatorStorage.sol


16      abstract contract OperatorDelegatorStorageV1 is IOperatorDelegator {
46      abstract contract OperatorDelegatorStorageV2 is OperatorDelegatorStorageV1 {
51      abstract contract OperatorDelegatorStorageV3 is OperatorDelegatorStorageV2 {
59      abstract contract OperatorDelegatorStorageV4 is OperatorDelegatorStorageV3 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegatorStorage.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueueStorage.sol


9       abstract contract DepositQueueStorageV1 is IDepositQueue {
26      abstract contract DepositQueueStorageV2 is DepositQueueStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueueStorage.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShimStorage.sol


7       abstract contract WBETHShimStorageV1 is AggregatorV3Interface {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShimStorage.sol


7       abstract contract METHShimStorageV1 is AggregatorV3Interface {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracleStorage.sol


8       abstract contract RenzoOracleStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracleStorage.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProviderStorage.sol


7       abstract contract BalancerRateProviderStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProviderStorage.sol#L0:0

```solidity
File: contracts/RestakeManagerStorage.sol


15      abstract contract RestakeManagerStorageV1 is IRestakeManager {
64      abstract contract RestakeManagerStorageV2 is RestakeManagerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandlerStorage.sol


6       abstract contract RewardHandlerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandlerStorage.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueueStorage.sol


9       abstract contract WithdrawQueueStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueueStorage.sol#L0:0

</details>

## NC078 - Abstract contract declarations should have NatSpec @notice annotations:

Explain to an end user what this does


<details>
<summary>Click to show 12 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridgeStorage.sol


18      abstract contract xRenzoBridgeStorageV1 is IxRenzoBridge {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridgeStorage.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol


7       abstract contract RenzoOracleL2StorageV1 is IRenzoOracleL2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


9       abstract contract xRenzoDepositStorageV1 is IxRenzoDeposit {
47      abstract contract xRenzoDepositStorageV2 is xRenzoDepositStorageV1 {
52      abstract contract xRenzoDepositStorageV3 is xRenzoDepositStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegatorStorage.sol


16      abstract contract OperatorDelegatorStorageV1 is IOperatorDelegator {
46      abstract contract OperatorDelegatorStorageV2 is OperatorDelegatorStorageV1 {
51      abstract contract OperatorDelegatorStorageV3 is OperatorDelegatorStorageV2 {
59      abstract contract OperatorDelegatorStorageV4 is OperatorDelegatorStorageV3 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegatorStorage.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueueStorage.sol


9       abstract contract DepositQueueStorageV1 is IDepositQueue {
26      abstract contract DepositQueueStorageV2 is DepositQueueStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueueStorage.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShimStorage.sol


7       abstract contract WBETHShimStorageV1 is AggregatorV3Interface {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShimStorage.sol


7       abstract contract METHShimStorageV1 is AggregatorV3Interface {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracleStorage.sol


8       abstract contract RenzoOracleStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracleStorage.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProviderStorage.sol


7       abstract contract BalancerRateProviderStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProviderStorage.sol#L0:0

```solidity
File: contracts/RestakeManagerStorage.sol


15      abstract contract RestakeManagerStorageV1 is IRestakeManager {
64      abstract contract RestakeManagerStorageV2 is RestakeManagerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandlerStorage.sol


6       abstract contract RewardHandlerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandlerStorage.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueueStorage.sol


9       abstract contract WithdrawQueueStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueueStorage.sol#L0:0

</details>

## NC079 - Abstract contract declarations should have Natspec @dev annotations:

Explain to a developer any extra details


<details>
<summary>Click to show 12 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridgeStorage.sol


18      abstract contract xRenzoBridgeStorageV1 is IxRenzoBridge {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridgeStorage.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol


7       abstract contract RenzoOracleL2StorageV1 is IRenzoOracleL2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


9       abstract contract xRenzoDepositStorageV1 is IxRenzoDeposit {
47      abstract contract xRenzoDepositStorageV2 is xRenzoDepositStorageV1 {
52      abstract contract xRenzoDepositStorageV3 is xRenzoDepositStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegatorStorage.sol


46      abstract contract OperatorDelegatorStorageV2 is OperatorDelegatorStorageV1 {
51      abstract contract OperatorDelegatorStorageV3 is OperatorDelegatorStorageV2 {
59      abstract contract OperatorDelegatorStorageV4 is OperatorDelegatorStorageV3 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegatorStorage.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueueStorage.sol


9       abstract contract DepositQueueStorageV1 is IDepositQueue {
26      abstract contract DepositQueueStorageV2 is DepositQueueStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueueStorage.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShimStorage.sol


7       abstract contract WBETHShimStorageV1 is AggregatorV3Interface {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShimStorage.sol


7       abstract contract METHShimStorageV1 is AggregatorV3Interface {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracleStorage.sol


8       abstract contract RenzoOracleStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracleStorage.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProviderStorage.sol


7       abstract contract BalancerRateProviderStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProviderStorage.sol#L0:0

```solidity
File: contracts/RestakeManagerStorage.sol


15      abstract contract RestakeManagerStorageV1 is IRestakeManager {
64      abstract contract RestakeManagerStorageV2 is RestakeManagerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandlerStorage.sol


6       abstract contract RewardHandlerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandlerStorage.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueueStorage.sol


9       abstract contract WithdrawQueueStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueueStorage.sol#L0:0

</details>

## NC080 - Modifier definitions should have Natspec @notice annotations:

Explain to an end user what this does


<details>
<summary>Click to show 10 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


50          modifier onlyBridgeAdmin() {
55          modifier onlyPriceFeedSender() {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


43          modifier onlySource(address _originSender, uint32 _origin) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


50          modifier onlyOperatorDelegatorAdmin() {
56          modifier onlyRestakeManager() {
62          modifier onlyNativeEthRestakeAdmin() {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


44          modifier onlyRestakeManagerAdmin() {
50          modifier onlyRestakeManager() {
56          modifier onlyNativeEthRestakeAdmin() {
62          modifier onlyERC20RewardsAdmin() {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


29          modifier onlyOracleAdmin() {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


71          modifier onlyRestakeManagerAdmin() {
77          modifier onlyDepositWithdrawPauserAdmin() {
83          modifier onlyDepositQueue() {
89          modifier notPaused() {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


18          modifier onlyNativeEthRestakeAdmin() {
24          modifier onlyRestakeManagerAdmin() {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


127         modifier onlyRoleOrOpenRole(bytes32 role) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


39          modifier onlyWithdrawQueueAdmin() {
45          modifier onlyRestakeManager() {
50          modifier onlyDepositQueue() {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


15          modifier onlyMinterBurner() {
21          modifier onlyTokenAdmin() {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## NC081 - Modifier definitions should have Natspec @dev annotations:

Explain to a developer any extra details


```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


50          modifier onlyBridgeAdmin() {
55          modifier onlyPriceFeedSender() {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


43          modifier onlySource(address _originSender, uint32 _origin) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


50          modifier onlyDepositQueue() {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

## NC082 - Contract definitions should have Natspec @title annotations:

 title that should describe the contract


<details>
<summary>Click to show 21 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


30      contract LockboxAdapterBlast {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


16      contract xRenzoBridge is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


11      contract RenzoOracleL2 is Initializable, OwnableUpgradeable, RenzoOracleL2StorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


10      contract ConnextReceiver is IXReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


27      contract xRenzoDeposit is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


16      contract XERC20 is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


14      contract XERC20Factory is Initializable, IXERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


11      contract XERC20Lockbox is Initializable, IXERC20Lockbox {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


11      contract OptimismMintableXERC20 is ERC165Upgradeable, XERC20, IOptimismMintableERC20 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


14      contract OptimismMintableXERC20Factory is Initializable, XERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


17      contract OperatorDelegator is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


10      contract DepositQueue is Initializable, ReentrancyGuardUpgradeable, DepositQueueStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


15      contract WBETHShim is Initializable, WBETHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


15      contract METHShim is Initializable, METHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


13      contract RenzoOracle is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManagerStorage.sol


37      contract RoleManagerStorageV2 is RoleManagerStorageV1 {
45      contract RoleManagerStorageV3 is RoleManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManagerStorage.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


9       contract BalancerRateProvider is Initializable, IRateProvider, BalancerRateProviderStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


26      contract RestakeManager is Initializable, ReentrancyGuardUpgradeable, RestakeManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/TimelockController.sol


25      contract TimelockController is AccessControl, IERC721Receiver, IERC1155Receiver {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


11      contract WithdrawQueue is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


13      contract EzEthToken is Initializable, ERC20Upgradeable, IEzEthToken, EzEthTokenStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## NC083 - Contract definitions should have Natspec @author annotations:

The name of the author


<details>
<summary>Click to show 24 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


30      contract LockboxAdapterBlast {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


16      contract xRenzoBridge is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


11      contract RenzoOracleL2 is Initializable, OwnableUpgradeable, RenzoOracleL2StorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


14      contract Receiver is CCIPReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


10      contract ConnextReceiver is IXReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


27      contract xRenzoDeposit is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


16      contract XERC20 is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


14      contract XERC20Factory is Initializable, IXERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


11      contract XERC20Lockbox is Initializable, IXERC20Lockbox {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


11      contract OptimismMintableXERC20 is ERC165Upgradeable, XERC20, IOptimismMintableERC20 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


14      contract OptimismMintableXERC20Factory is Initializable, XERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


17      contract OperatorDelegator is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


10      contract DepositQueue is Initializable, ReentrancyGuardUpgradeable, DepositQueueStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


15      contract WBETHShim is Initializable, WBETHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


15      contract METHShim is Initializable, METHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


13      contract RenzoOracle is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


14      contract RoleManager is IRoleManager, AccessControlUpgradeable, RoleManagerStorageV3 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/Permissions/RoleManagerStorage.sol


9       contract RoleManagerStorageV1 {
37      contract RoleManagerStorageV2 is RoleManagerStorageV1 {
45      contract RoleManagerStorageV3 is RoleManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManagerStorage.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


9       contract BalancerRateProvider is Initializable, IRateProvider, BalancerRateProviderStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


26      contract RestakeManager is Initializable, ReentrancyGuardUpgradeable, RestakeManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/TimelockController.sol


25      contract TimelockController is AccessControl, IERC721Receiver, IERC1155Receiver {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


11      contract WithdrawQueue is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


13      contract EzEthToken is Initializable, ERC20Upgradeable, IEzEthToken, EzEthTokenStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

```solidity
File: contracts/token/EzEthTokenStorage.sol


10      contract EzEthTokenStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthTokenStorage.sol#L0:0

</details>

## NC084 - Contract definitions should have Natspec @notice annotations:

Explain to an end user what this does


<details>
<summary>Click to show 23 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


16      contract xRenzoBridge is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


11      contract RenzoOracleL2 is Initializable, OwnableUpgradeable, RenzoOracleL2StorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


14      contract Receiver is CCIPReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


10      contract ConnextReceiver is IXReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


27      contract xRenzoDeposit is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


16      contract XERC20 is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


14      contract XERC20Factory is Initializable, IXERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


11      contract XERC20Lockbox is Initializable, IXERC20Lockbox {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


11      contract OptimismMintableXERC20 is ERC165Upgradeable, XERC20, IOptimismMintableERC20 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


14      contract OptimismMintableXERC20Factory is Initializable, XERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


17      contract OperatorDelegator is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


10      contract DepositQueue is Initializable, ReentrancyGuardUpgradeable, DepositQueueStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


15      contract WBETHShim is Initializable, WBETHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


15      contract METHShim is Initializable, METHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


13      contract RenzoOracle is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


14      contract RoleManager is IRoleManager, AccessControlUpgradeable, RoleManagerStorageV3 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/Permissions/RoleManagerStorage.sol


9       contract RoleManagerStorageV1 {
37      contract RoleManagerStorageV2 is RoleManagerStorageV1 {
45      contract RoleManagerStorageV3 is RoleManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManagerStorage.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


9       contract BalancerRateProvider is Initializable, IRateProvider, BalancerRateProviderStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


26      contract RestakeManager is Initializable, ReentrancyGuardUpgradeable, RestakeManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/TimelockController.sol


25      contract TimelockController is AccessControl, IERC721Receiver, IERC1155Receiver {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


11      contract WithdrawQueue is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


13      contract EzEthToken is Initializable, ERC20Upgradeable, IEzEthToken, EzEthTokenStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

```solidity
File: contracts/token/EzEthTokenStorage.sol


10      contract EzEthTokenStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthTokenStorage.sol#L0:0

</details>

## NC085 - Contract definitions should have Natspec @dev annotations:

Explain to a developer any extra details


<details>
<summary>Click to show 17 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


16      contract xRenzoBridge is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


11      contract RenzoOracleL2 is Initializable, OwnableUpgradeable, RenzoOracleL2StorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


14      contract Receiver is CCIPReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


10      contract ConnextReceiver is IXReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


27      contract xRenzoDeposit is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


16      contract XERC20 is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


14      contract XERC20Factory is Initializable, IXERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


11      contract XERC20Lockbox is Initializable, IXERC20Lockbox {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


11      contract OptimismMintableXERC20 is ERC165Upgradeable, XERC20, IOptimismMintableERC20 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


14      contract OptimismMintableXERC20Factory is Initializable, XERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


10      contract DepositQueue is Initializable, ReentrancyGuardUpgradeable, DepositQueueStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


15      contract WBETHShim is Initializable, WBETHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


15      contract METHShim is Initializable, METHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Permissions/RoleManagerStorage.sol


37      contract RoleManagerStorageV2 is RoleManagerStorageV1 {
45      contract RoleManagerStorageV3 is RoleManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManagerStorage.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


9       contract BalancerRateProvider is Initializable, IRateProvider, BalancerRateProviderStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


26      contract RestakeManager is Initializable, ReentrancyGuardUpgradeable, RestakeManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


11      contract WithdrawQueue is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## NC086 - Event definitions should have Natspec @notice annotations:

Explain to an end user what this does


<details>
<summary>Click to show 12 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


25          event EzETHMinted(
34          event MessageSent(
43          event ConnextMessageSent(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


15          event OracleAddressUpdated(address newOracle, address oldOracle);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


24          event XRenzoBridgeL1Updated(address newBridgeAddress, address oldBridgeAddress);
25          event CCIPEthChainSelectorUpdated(uint64 newSourceChainSelector, uint64 oldSourceChainSelector);
26          event XRenzoDepositUpdated(address newRenzoDeposit, address oldRenzoDeposit);
35          event MessageReceived(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


23          event XRenzoBridgeL1Updated(address newBridgeAddress, address oldBridgeAddress);
24          event ConnextEthChainDomainUpdated(uint32 newSourceChainDomain, uint32 oldSourceChainDomain);
25          event XRenzoDepositUpdated(address newRenzoDeposit, address oldRenzoDeposit);
35          event MessageReceived(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


42          event PriceUpdated(uint256 price, uint256 timestamp);
43          event Deposit(address indexed user, uint256 amountIn, uint256 amountOut);
44          event BridgeSweeperAddressUpdated(address sweeper, bool allowed);
45          event BridgeSwept(
51          event OraclePriceFeedUpdated(address newOracle, address oldOracle);
52          event ReceiverPriceFeedUpdated(address newReceiver, address oldReceiver);
53          event SweeperBridgeFeeCollected(address sweeper, uint256 feeCollected);
54          event BridgeFeeShareUpdated(uint256 oldBridgeFeeShare, uint256 newBridgeFeeShare);
55          event SweepBatchSizeUpdated(uint256 oldSweepBatchSize, uint256 newSweepBatchSize);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


28          event TokenStrategyUpdated(IERC20 token, IStrategy strategy);
29          event DelegationAddressUpdated(address delegateAddress);
30          event RewardsForwarded(address rewardDestination, uint256 amount);
32          event WithdrawStarted(
43          event WithdrawCompleted(bytes32 withdrawalRoot, IStrategy[] strategies, uint256[] shares);
45          event GasSpent(address admin, uint256 gasSpent);
46          event GasRefunded(address admin, uint256 gasRefunded);
47          event BaseGasAmountSpentUpdated(uint256 oldBaseGasAmountSpent, uint256 newBaseGasAmountSpent);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


15          event RewardsDeposited(IERC20 token, uint256 amount);
17          event FeeConfigUpdated(address feeAddress, uint256 feeBasisPoints);
19          event RestakeManagerUpdated(IRestakeManager restakeManager);
21          event ETHDepositedFromProtocol(uint256 amount);
23          event ETHStakedFromQueue(
30          event ProtocolFeesPaid(IERC20 token, uint256 amount, address destination);
32          event GasRefunded(address admin, uint256 gasRefunded);
35          event WithdrawQueueUpdated(address oldWithdrawQueue, address newWithdrawQueue);
38          event BufferFilled(address token, uint256 amount);
41          event FullWithdrawalETHReceived(uint256 amount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


35          event OracleAddressUpdated(IERC20 token, AggregatorV3Interface oracleAddress);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


30          event OperatorDelegatorAdded(IOperatorDelegator od);
31          event OperatorDelegatorRemoved(IOperatorDelegator od);
32          event OperatorDelegatorAllocationUpdated(IOperatorDelegator od, uint256 allocation);
34          event CollateralTokenAdded(IERC20 token);
35          event CollateralTokenRemoved(IERC20 token);
41          event Deposit(
50          event UserWithdrawStarted(
59          event UserWithdrawCompleted(
68          event CollateralTokenTvlUpdated(IERC20 token, uint256 tvl);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


29          event RewardDestinationUpdated(address rewardDestination);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


38          event CallScheduled(
51          event CallExecuted(
62          event CallSalt(bytes32 indexed id, bytes32 salt);
67          event Cancelled(bytes32 indexed id);
72          event MinDelayChange(uint256 oldDuration, uint256 newDuration);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


19          event WithdrawBufferTargetUpdated(uint256 oldBufferTarget, uint256 newBufferTarget);
21          event CoolDownPeriodUpdated(uint256 oldCoolDownPeriod, uint256 newCoolDownPeriod);
23          event EthBufferFilled(uint256 amount);
25          event ERC20BufferFilled(address asset, uint256 amount);
27          event WithdrawRequestCreated(
36          event WithdrawRequestClaimed(WithdrawRequest withdrawRequest);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## NC087 - State variable declarations should have Natspec @notice annotations:

Explain to an end user what this does


<details>
<summary>Click to show 23 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


31          address immutable blastStandardBridge;
32          address immutable registry;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


61          uint8 public constant EXPECTED_DECIMALS = 18;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


13          uint256 public constant MAX_TIME_WINDOW = 86400 + 60; // 24 hours + 60 seconds


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol


8           AggregatorV3Interface public oracle;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


37          uint8 public constant EXPECTED_DECIMALS = 18;
40          uint32 public constant FEE_BASIS = 10000;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


54          uint256 public bridgeFeeShare;
57          uint256 public sweepBatchSize;
60          uint256 public bridgeFeeCollected;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


29          bool public IS_NATIVE;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


24          uint256 internal constant GWEI_TO_WEI = 1e9;
26          address public constant IS_NATIVE = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegatorStorage.sol


18          IRoleManager public roleManager;
21          IStrategyManager public strategyManager;
24          IRestakeManager public restakeManager;
28          mapping(IERC20 => IStrategy) public tokenStrategyMapping;
31          address public delegateAddress;
34          IDelegationManager public delegationManager;
37          IEigenPodManager public eigenPodManager;
40          IEigenPod public eigenPod;
43          uint256 public stakedButNotVerifiedEth;
48          uint256 public pendingUnstakedDelayedWithdrawalAmount;
53          uint256 public baseGasAmountSpent;
56          mapping(address => uint256) public adminGasSpentInWei;
61          mapping(address => uint256) public queuedShares;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegatorStorage.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


13          address public constant IS_NATIVE = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueueStorage.sol


11          IRoleManager public roleManager;
14          IRestakeManager public restakeManager;
17          address public feeAddress;
20          uint256 public feeBasisPoints;
23          mapping(address => uint256) public totalEarned;
27          IWithdrawQueue public withdrawQueue;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueueStorage.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShimStorage.sol


8           IStakedTokenV2 public wBETHToken;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShimStorage.sol


8           IMethStaking public methStaking;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


20          string constant INVALID_0_INPUT = "Invalid 0 input";
23          uint256 constant SCALE_FACTOR = 10 ** 18;
26          uint256 constant MAX_TIME_WINDOW = 86400 + 60; // 24 hours + 60 seconds


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracleStorage.sol


10          IRoleManager public roleManager;
13          mapping(IERC20 => AggregatorV3Interface) public tokenOracleLookup;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracleStorage.sol#L0:0

```solidity
File: contracts/Permissions/RoleManagerStorage.sol


11          bytes32 public constant RX_ETH_MINTER_BURNER = keccak256("RX_ETH_MINTER_BURNER");
14          bytes32 public constant OPERATOR_DELEGATOR_ADMIN = keccak256("OPERATOR_DELEGATOR_ADMIN");
17          bytes32 public constant ORACLE_ADMIN = keccak256("ORACLE_ADMIN");
20          bytes32 public constant RESTAKE_MANAGER_ADMIN = keccak256("RESTAKE_MANAGER_ADMIN");
23          bytes32 public constant TOKEN_ADMIN = keccak256("TOKEN_ADMIN");
26          bytes32 public constant NATIVE_ETH_RESTAKE_ADMIN = keccak256("NATIVE_ETH_RESTAKE_ADMIN");
29          bytes32 public constant ERC20_REWARD_ADMIN = keccak256("ERC20_REWARD_ADMIN");
32          bytes32 public constant DEPOSIT_WITHDRAW_PAUSER = keccak256("DEPOSIT_WITHDRAW_PAUSER");
39          bytes32 public constant BRIDGE_ADMIN = keccak256("BRIDGE_ADMIN");
42          bytes32 public constant PRICE_FEED_SENDER = keccak256("PRICE_FEED_SENDER");
47          bytes32 public constant WITHDRAW_QUEUE_ADMIN = keccak256("WITHDRAW_QUEUE_ADMIN");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManagerStorage.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProviderStorage.sol


9           IRestakeManager public restakeManager;
12          IERC20Upgradeable public ezETHToken;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProviderStorage.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


38          uint256 constant BASIS_POINTS = 100;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/RestakeManagerStorage.sol


17          IRoleManager public roleManager;
20          IEzEthToken public ezETH;
23          IStrategyManager public strategyManager;
26          IDelegationManager public delegationManager;
39          mapping(bytes32 => PendingWithdrawal) public pendingWithdrawals;
42          IOperatorDelegator[] public operatorDelegators;
46          mapping(IOperatorDelegator => uint256) public operatorDelegatorAllocations;
49          IERC20[] public collateralTokens;
52          IRenzoOracle public renzoOracle;
55          bool public paused;
58          uint256 public maxDepositTVL;
61          IDepositQueue public depositQueue;
65          mapping(IERC20 => uint256) public collateralTokenTvlLimits;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandlerStorage.sol


8           IRoleManager public roleManager;
11          address public rewardDestination;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandlerStorage.sol#L0:0

```solidity
File: contracts/TimelockController.sol


26          bytes32 public constant TIMELOCK_ADMIN_ROLE = keccak256("TIMELOCK_ADMIN_ROLE");
27          bytes32 public constant PROPOSER_ROLE = keccak256("PROPOSER_ROLE");
28          bytes32 public constant EXECUTOR_ROLE = keccak256("EXECUTOR_ROLE");
29          bytes32 public constant CANCELLER_ROLE = keccak256("CANCELLER_ROLE");
30          uint256 internal constant _DONE_TIMESTAMP = uint256(1);
32          mapping(bytes32 => uint256) private _timestamps;
33          uint256 private _minDelay;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueueStorage.sol


10          address public constant IS_NATIVE = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;
26          IRenzoOracle public renzoOracle;
29          IEzEthToken public ezETH;
32          IRoleManager public roleManager;
35          IRestakeManager public restakeManager;
38          uint256 public coolDownPeriod;
41          uint256 public withdrawRequestNonce;
44          mapping(address => uint256) public withdrawalBufferTarget;
47          mapping(address => uint256) public claimReserve;
50          mapping(address => WithdrawRequest[]) public withdrawRequests;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueueStorage.sol#L0:0

```solidity
File: contracts/token/EzEthTokenStorage.sol


12          IRoleManager public roleManager;
15          bool public paused;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthTokenStorage.sol#L0:0

</details>

## NC088 - State variable declarations should have Natspec @dev annotations:

Explain to a developer any extra details


<details>
<summary>Click to show 19 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


31          address immutable blastStandardBridge;
32          address immutable registry;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridgeStorage.sol


20          IERC20 public xezETH;
23          IERC20 public ezETH;
26          IRestakeManager public restakeManager;
29          IERC20 public wETH;
32          IXERC20Lockbox public xezETHLockbox;
35          IConnext public connext;
38          IRateProvider public rateProvider;
41          IRouterClient public linkRouterClient;
44          LinkTokenInterface public linkToken;
47          IRoleManager public roleManager;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridgeStorage.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol


8           AggregatorV3Interface public oracle;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2Storage.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


16          address public xRenzoBridgeL1;
19          uint64 public ccipEthChainSelector;
22          IxRenzoDeposit public xRenzoDeposit;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


12          address public connext;
15          address public xRenzoBridgeL1;
18          uint32 public connextEthChainDomain;
21          IxRenzoDeposit public xRenzoDeposit;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


11          uint256 public lastPriceTimestamp;
14          uint256 public lastPrice;
17          IERC20 public xezETH;
20          IERC20 public depositToken;
23          IERC20 public collateralToken;
26          IConnext public connext;
29          bytes32 public swapKey;
32          address public receiver;
35          uint256 public bridgeRouterFeeBps;
38          uint32 public bridgeDestinationDomain;
41          address public bridgeTargetAddress;
44          mapping(address => bool) public allowedBridgeSweepers;
49          IRenzoOracleL2 public oracle;
54          uint256 public bridgeFeeShare;
57          uint256 public sweepBatchSize;
60          uint256 public bridgeFeeCollected;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


26          uint256 private constant _DURATION = 1 days;
31          address public FACTORY;
36          address public lockbox;
41          mapping(address => Bridge) public bridges;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


20          mapping(address => address) internal _lockboxRegistry;
25          EnumerableSetUpgradeable.AddressSet internal _lockboxRegistryArray;
30          EnumerableSetUpgradeable.AddressSet internal _xerc20RegistryArray;
35          address public lockboxImplementation;
40          address public xerc20Implementation;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


18          IXERC20 public XERC20;
23          IERC20 public ERC20;
29          bool public IS_NATIVE;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


15          address public l1Token;
20          address public optimismBridge;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


24          uint256 internal constant GWEI_TO_WEI = 1e9;
26          address public constant IS_NATIVE = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


13          address public constant IS_NATIVE = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueueStorage.sol


27          IWithdrawQueue public withdrawQueue;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueueStorage.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShimStorage.sol


8           IStakedTokenV2 public wBETHToken;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShimStorage.sol


8           IMethStaking public methStaking;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShimStorage.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


23          uint256 constant SCALE_FACTOR = 10 ** 18;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RestakeManagerStorage.sol


65          mapping(IERC20 => uint256) public collateralTokenTvlLimits;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L0:0

```solidity
File: contracts/TimelockController.sol


26          bytes32 public constant TIMELOCK_ADMIN_ROLE = keccak256("TIMELOCK_ADMIN_ROLE");
27          bytes32 public constant PROPOSER_ROLE = keccak256("PROPOSER_ROLE");
28          bytes32 public constant EXECUTOR_ROLE = keccak256("EXECUTOR_ROLE");
29          bytes32 public constant CANCELLER_ROLE = keccak256("CANCELLER_ROLE");
30          uint256 internal constant _DONE_TIMESTAMP = uint256(1);
32          mapping(bytes32 => uint256) private _timestamps;
33          uint256 private _minDelay;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueueStorage.sol


10          address public constant IS_NATIVE = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueueStorage.sol#L0:0

</details>

## NC089 - Functions should have Natspec @return annotations:

Documents the return variables of a contract’s function


<details>
<summary>Click to show 19 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


10          function getXERC20(address erc20) external view returns (address xerc20);
12          function getERC20(address xerc20) external view returns (address erc20);
14          function getLockbox(address erc20) external view returns (address xerc20);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


139         function xReceive(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


50          function getMintRate() public view returns (uint256, uint256) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


69          function xReceive(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


277         function getBridgeFeeShare(uint256 _amountIn) public view returns (uint256) {
289         function getMintRate() public view returns (uint256, uint256) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


152         function mintingMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
163         function burningMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
174         function mintingCurrentLimitOf(address _bridge) public view returns (uint256 _limit) {
190         function burningCurrentLimitOf(address _bridge) public view returns (uint256 _limit) {
276         function _calculateNewCurrentLimit(
302         function _getCurrentLimit(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


74          function deployXERC20(
105         function deployLockbox(
135         function _deployXERC20(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


48          function supportsInterface(
56          function remoteToken() public view override returns (address) {
60          function bridge() public view override returns (address) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


37          function deployOptimismMintableXERC20(
73          function _deployOptimismMintableXERC20(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


172         function getStrategyIndex(IStrategy _strategy) public view returns (uint256) {
327         function getTokenBalanceFromStrategy(IERC20 token) external view returns (uint256) {
338         function getStakedETHBalance() external view returns (uint256) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/IRestakeManager.sol


15          function depositQueue() external view returns (IDepositQueue);
17          function calculateTVLs() external view returns (uint256[][] memory, uint256[] memory, uint256);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/IRestakeManager.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


29          function decimals() external pure returns (uint8) {
33          function description() external pure returns (string memory) {
37          function version() external pure returns (uint256) {
42          function getRoundData(uint80) external pure returns (uint80, int256, uint256, uint256, uint80) {
46          function latestRoundData()


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


29          function decimals() external pure returns (uint8) {
33          function description() external pure returns (string memory) {
37          function version() external pure returns (uint256) {
42          function getRoundData(uint80) external pure returns (uint80, int256, uint256, uint256, uint80) {
46          function latestRoundData()


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


71          function lookupTokenValue(IERC20 _token, uint256 _balance) public view returns (uint256) {
85          function lookupTokenAmountFromValue(
103         function lookupTokenValues(
123         function calculateMintAmount(
152         function calculateRedeemAmount(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


32          function isRoleManagerAdmin(address potentialAddress) external view returns (bool) {
38          function isEzETHMinterBurner(address potentialAddress) external view returns (bool) {
44          function isOperatorDelegatorAdmin(address potentialAddress) external view returns (bool) {
50          function isOracleAdmin(address potentialAddress) external view returns (bool) {
56          function isRestakeManagerAdmin(address potentialAddress) external view returns (bool) {
62          function isTokenAdmin(address potentialAddress) external view returns (bool) {
68          function isNativeEthRestakeAdmin(address potentialAddress) external view returns (bool) {
74          function isERC20RewardsAdmin(address potentialAddress) external view returns (bool) {
80          function isDepositWithdrawPauser(address potentialAddress) external view returns (bool) {
86          function isBridgeAdmin(address potentialAddress) external view returns (bool) {
92          function isPriceFeedSender(address potentialAddress) external view returns (bool) {
98          function isWithdrawQueueAdmin(address potentialAddress) external view returns (bool) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


29          function getRate() external view returns (uint256) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


126         function getOperatorDelegatorsLength() external view returns (uint256) {
266         function getCollateralTokensLength() external view returns (uint256) {
451         function getCollateralTokenIndex(IERC20 _collateralToken) public view returns (uint256) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/TimelockController.sol


142         function supportsInterface(
154         function isOperation(bytes32 id) public view virtual returns (bool) {
161         function isOperationPending(bytes32 id) public view virtual returns (bool) {
168         function isOperationReady(bytes32 id) public view virtual returns (bool) {
176         function isOperationDone(bytes32 id) public view virtual returns (bool) {
184         function getTimestamp(bytes32 id) public view virtual returns (uint256) {
193         function getMinDelay() public view virtual returns (uint256) {
201         function hashOperation(
215         function hashOperationBatch(
411         function onERC721Received(
423         function onERC1155Received(
436         function onERC1155BatchReceived(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


77          function name() public view virtual override returns (string memory) {
85          function symbol() public view virtual override returns (string memory) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## NC090 - Functions should have Natspec @param annotations:

Documents a parameter just like in Doxygen (must be followed by parameter name)


<details>
<summary>Click to show 23 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


39          constructor(address _blastStandardBridge, address _registry) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


65          constructor() {
70          function initialize(
139         function xReceive(
313         receive() external payable {}


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


19          constructor() {
23          function initialize(AggregatorV3Interface _oracle) public initializer {
36          function setOracleAddress(AggregatorV3Interface _oracleAddress) external onlyOwner {
50          function getMintRate() public view returns (uint256, uint256) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


43          constructor(
117         function unPause() external onlyOwner {
125         function pause() external onlyOwner {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


52          constructor(address _connext, address _xRenzoBridgeL1, uint32 _connextEthChainDomain) {
69          function xReceive(
113         function unPause() external onlyOwner {
121         function pause() external onlyOwner {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


59          constructor() {
289         function getMintRate() public view returns (uint256, uint256) {
396         function _recoverBridgeFee() internal {
414         function sweep() public payable nonReentrant {
456         function getRate() external view override returns (uint256) {
542         receive() external payable {}


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


50          constructor() {
96          function mint(address _user, uint256 _amount) public virtual {
107         function burn(address _user, uint256 _amount) public virtual {
121         function setLockbox(address _lockbox) public {
152         function mintingMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
163         function burningMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
174         function mintingCurrentLimitOf(address _bridge) public view returns (uint256 _limit) {
190         function burningCurrentLimitOf(address _bridge) public view returns (uint256 _limit) {
205         function _useMinterLimits(address _bridge, uint256 _change) internal {
217         function _useBurnerLimits(address _bridge, uint256 _change) internal {
230         function _changeMinterLimit(address _bridge, uint256 _limit) internal {
252         function _changeBurnerLimit(address _bridge, uint256 _limit) internal {
276         function _calculateNewCurrentLimit(
302         function _getCurrentLimit(
328         function _burnWithCaller(address _caller, address _user, uint256 _amount) internal {
348         function _mintWithCaller(address _caller, address _user, uint256 _amount) internal {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


44          constructor() {
74          function deployXERC20(
105         function deployLockbox(
135         function _deployXERC20(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


33          constructor() {
54          function depositNative() public payable {
66          function deposit(uint256 _amount) external {
79          function depositTo(address _to, uint256 _amount) external {
91          function depositNativeTo(address _to) public payable {
103         function withdraw(uint256 _amount) external {
114         function withdrawTo(address _to, uint256 _amount) external {
125         function _withdraw(address _to, uint256 _amount) internal {
145         function _deposit(address _to, uint256 _amount) internal {
157         receive() external payable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


24          constructor() {
48          function supportsInterface(
56          function remoteToken() public view override returns (address) {
60          function bridge() public view override returns (address) {
64          function mint(address _to, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
68          function burn(address _from, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


19          constructor() {
37          function deployOptimismMintableXERC20(
73          function _deployOptimismMintableXERC20(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


69          constructor() {
74          function initialize(
101         function activateRestaking() external nonReentrant onlyNativeEthRestakeAdmin {
106         function setTokenStrategy(
117         function setDelegateAddress(
132         function setBaseGasAmountSpent(
143         function deposit(
172         function getStrategyIndex(IStrategy _strategy) public view returns (uint256) {
327         function getTokenBalanceFromStrategy(IERC20 token) external view returns (uint256) {
338         function getStakedETHBalance() external view returns (uint256) {
349         function stakeEth(
364         function verifyWithdrawalCredentials(
459         function startDelayedWithdrawUnstakedETH() external onlyNativeEthRestakeAdmin {
481         function _refundGas() internal returns (uint256) {
501         receive() external payable nonReentrant {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


69          constructor() {
74          function initialize(IRoleManager _roleManager) public initializer {
93          function setFeeConfig(
112         function setRestakeManager(IRestakeManager _restakeManager) external onlyRestakeManagerAdmin {
123         function depositETHFromProtocol() external payable onlyRestakeManager {
152         function forwardFullWithdrawalETH() external payable nonReentrant {
163         receive() external payable nonReentrant {
187         function stakeEthFromQueue(
211         function stakeEthFromQueueMulti(
254         function sweepERC20(IERC20 token) external onlyERC20RewardsAdmin {
294         function _checkAndFillETHWithdrawBuffer(uint256 _amount) internal {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


18          constructor() {
23          function initialize(IStakedTokenV2 _wBETHToken) public initializer {
29          function decimals() external pure returns (uint8) {
33          function description() external pure returns (string memory) {
37          function version() external pure returns (uint256) {
42          function getRoundData(uint80) external pure returns (uint80, int256, uint256, uint256, uint80) {
46          function latestRoundData()
69          function _getWBETHData()


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


18          constructor() {
23          function initialize(IMethStaking _methStaking) public initializer {
29          function decimals() external pure returns (uint8) {
33          function description() external pure returns (string memory) {
37          function version() external pure returns (uint256) {
42          function getRoundData(uint80) external pure returns (uint80, int256, uint256, uint256, uint80) {
46          function latestRoundData()
69          function _getMETHData()


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


39          constructor() {
44          function initialize(IRoleManager _roleManager) public initializer {
54          function setOracleAddress(
71          function lookupTokenValue(IERC20 _token, uint256 _balance) public view returns (uint256) {
85          function lookupTokenAmountFromValue(
103         function lookupTokenValues(
123         function calculateMintAmount(
152         function calculateRedeemAmount(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


17          constructor() {
22          function initialize(address roleManagerAdmin) public initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


12          constructor() {
17          function initialize(
29          function getRate() external view returns (uint256) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


96          constructor() {
101         function initialize(
121         function setPaused(bool _paused) external onlyDepositWithdrawPauserAdmin {
126         function getOperatorDelegatorsLength() external view returns (uint256) {
131         function addOperatorDelegator(
160         function removeOperatorDelegator(
187         function setOperatorDelegatorAllocation(
215         function setMaxDepositTVL(uint256 _maxDepositTVL) external onlyRestakeManagerAdmin {
220         function addCollateralToken(IERC20 _newCollateralToken) external onlyRestakeManagerAdmin {
244         function removeCollateralToken(
266         function getCollateralTokensLength() external view returns (uint256) {
274         function calculateTVLs() public view returns (uint256[][] memory, uint256[] memory, uint256) {
362         function chooseOperatorDelegatorForDeposit(
400         function chooseOperatorDelegatorForWithdraw(
451         function getCollateralTokenIndex(IERC20 _collateralToken) public view returns (uint256) {
582         function depositETH() external payable {
620         function stakeEthInOperatorDelegator(
647         function depositTokenRewardsFromProtocol(
675         function getTotalRewardsEarned() external view returns (uint256) {
709         function setTokenTvlLimit(IERC20 _token, uint256 _limit) external onlyRestakeManagerAdmin {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


33          constructor() {
38          function initialize(IRoleManager _roleManager, address _rewardDestination) public initializer {
52          receive() external payable nonReentrant {
58          function forwardRewards() external nonReentrant onlyNativeEthRestakeAdmin {
62          function _forwardETH() internal {
72          function setRewardDestination(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


87          constructor(
137         receive() external payable {}
142         function supportsInterface(
154         function isOperation(bytes32 id) public view virtual returns (bool) {
161         function isOperationPending(bytes32 id) public view virtual returns (bool) {
168         function isOperationReady(bytes32 id) public view virtual returns (bool) {
176         function isOperationDone(bytes32 id) public view virtual returns (bool) {
184         function getTimestamp(bytes32 id) public view virtual returns (uint256) {
193         function getMinDelay() public view virtual returns (uint256) {
201         function hashOperation(
215         function hashOperationBatch(
234         function schedule(
259         function scheduleBatch(
283         function _schedule(bytes32 id, uint256 delay) private {
296         function cancel(bytes32 id) public virtual onlyRole(CANCELLER_ROLE) {
315         function execute(
342         function executeBatch(
368         function _execute(address target, uint256 value, bytes calldata data) internal virtual {
376         function _beforeCall(bytes32 id, bytes32 predecessor) private view {
387         function _afterCall(bytes32 id) private {
402         function updateDelay(uint256 newDelay) external virtual {
411         function onERC721Received(
423         function onERC1155Received(
436         function onERC1155BatchReceived(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


57          constructor() {
64          function initialize(
139         function pause() external onlyWithdrawQueueAdmin {
147         function unpause() external onlyWithdrawQueueAdmin {
182         function fillEthWithdrawBuffer() external payable nonReentrant onlyDepositQueue {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


28          constructor() {
33          function initialize(IRoleManager _roleManager) public initializer {
41          function mint(address to, uint256 amount) external onlyMinterBurner {
46          function burn(address from, uint256 amount) external onlyMinterBurner {
51          function setPaused(bool _paused) external onlyTokenAdmin {
56          function _beforeTokenTransfer(
77          function name() public view virtual override returns (string memory) {
85          function symbol() public view virtual override returns (string memory) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>


## NC091 - If statement control structures do not comply with best practices:

If statements which include a single line do not need to have curly brackets, however according to the Solidiity style guide the line of code executed upon the if statement condition being met should still be on the next line, not on the same line as the if statement declaration.


<details>
<summary>Click to show 19 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


51              if (!roleManager.isBridgeAdmin(msg.sender)) revert NotBridgeAdmin();
56              if (!roleManager.isPriceFeedSender(msg.sender)) revert NotPriceFeedSender();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


27              if (address(_oracle) == address(0)) revert InvalidZeroInput();
30              if (_oracle.decimals() > 18) revert InvalidTokenDecimals(18, _oracle.decimals());
37              if (address(_oracleAddress) == address(0)) revert InvalidZeroInput();
52              if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();
55              if (_scaledPrice < 1 ether) revert InvalidOraclePrice();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


48              if (_xRenzoBridgeL1 == address(0) || _ccipEthChainSelector == 0) revert InvalidZeroInput();
72              if (_ccipSender != xRenzoBridgeL1) revert InvalidSender(xRenzoBridgeL1, _ccipSender);
97              if (_newXRenzoBridgeL1 == address(0)) revert InvalidZeroInput();
108             if (_newChainSelector == 0) revert InvalidZeroInput();
135             if (_newXRenzoDeposit == address(0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


93              if (_newXRenzoBridgeL1 == address(0)) revert InvalidZeroInput();
104             if (_newChainDomain == 0) revert InvalidZeroInput();
131             if (_newXRenzoDeposit == address(0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


291             if (receiver == address(0) && address(oracle) == address(0)) revert PriceFeedNotAvailable();
311             if (msg.sender != receiver) revert InvalidSender(receiver, msg.sender);
404             if (!success) revert TransferFailed();
522             if (_newShare > 100) revert InvalidBridgeFeeShare(_newShare);
533             if (_newBatchSize < 32 ether) revert InvalidSweepBatchSize(_newBatchSize);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


122             if (msg.sender != FACTORY) revert IXERC20_NotFactory();
330             if (_amount == 0) revert IXERC20_INVALID_0_VALUE();
334                 if (_currentLimit < _amount) revert IXERC20_NotHighEnoughLimits();
350             if (_amount == 0) revert IXERC20_INVALID_0_VALUE();
354                 if (_currentLimit < _amount) revert IXERC20_NotHighEnoughLimits();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


115             if (XERC20(_xerc20).owner() != msg.sender) revert IXERC20Factory_NotOwner();
116             if (_lockboxRegistry[_xerc20] != address(0)) revert IXERC20Factory_LockboxAlreadyDeployed();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


55              if (!IS_NATIVE) revert IXERC20Lockbox_NotNative();
67              if (IS_NATIVE) revert IXERC20Lockbox_Native();
80              if (IS_NATIVE) revert IXERC20Lockbox_Native();
92              if (!IS_NATIVE) revert IXERC20Lockbox_NotNative();
132                 if (!_success) revert IXERC20Lockbox_WithdrawFailed();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


51              if (!roleManager.isOperatorDelegatorAdmin(msg.sender)) revert NotOperatorDelegatorAdmin();
57              if (msg.sender != address(restakeManager)) revert NotRestakeManager();
63              if (!roleManager.isNativeEthRestakeAdmin(msg.sender)) revert NotNativeEthRestakeAdmin();
81              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
82              if (address(_strategyManager) == address(0x0)) revert InvalidZeroInput();
83              if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
84              if (address(_delegationManager) == address(0x0)) revert InvalidZeroInput();
85              if (address(_eigenPodManager) == address(0x0)) revert InvalidZeroInput();
110             if (address(_token) == address(0x0)) revert InvalidZeroInput();
122             if (address(_delegateAddress) == address(0x0)) revert InvalidZeroInput();
123             if (address(delegateAddress) != address(0x0)) revert DelegateAddressAlreadySet();
135             if (_baseGasAmountSpent == 0) revert InvalidZeroInput();
199             if (tokens.length != tokenAmounts.length) revert MismatchedArrayLengths();
271             if (tokens.length != withdrawal.strategies.length) revert MismatchedArrayLengths();
278                 if (address(tokens[i]) == address(0)) revert InvalidZeroInput();
486             if (!success) revert TransferFailed();
521                 if (!success) revert TransferFailed();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


45              if (!roleManager.isRestakeManagerAdmin(msg.sender)) revert NotRestakeManagerAdmin();
51              if (msg.sender != address(restakeManager)) revert NotRestakeManager();
57              if (!roleManager.isNativeEthRestakeAdmin(msg.sender)) revert NotNativeEthRestakeAdmin();
63              if (!roleManager.isERC20RewardsAdmin(msg.sender)) revert NotERC20RewardsAdmin();
77              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
88              if (address(_withdrawQueue) == address(0)) revert InvalidZeroInput();
99                  if (_feeAddress == address(0x0)) revert InvalidZeroInput();
103             if (_feeBasisPoints > 10000) revert OverMaxBasisPoints();
113             if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
138             if (_amount == 0 || _asset == address(0)) revert InvalidZeroInput();
169                 if (!success) revert TransferFailed();
287             if (!success) revert TransferFailed();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


24              if (address(_wBETHToken) == address(0x0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


24              if (address(_methStaking) == address(0x0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


30              if (!roleManager.isOracleAdmin(msg.sender)) revert NotOracleAdmin();
45              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
58              if (address(_token) == address(0x0)) revert InvalidZeroInput();
73              if (address(oracle) == address(0x0)) revert OracleNotFound();
76              if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();
77              if (price <= 0) revert InvalidOraclePrice();
90              if (address(oracle) == address(0x0)) revert OracleNotFound();
93              if (timestamp < block.timestamp - MAX_TIME_WINDOW) revert OraclePriceExpired();
94              if (price <= 0) revert InvalidOraclePrice();
107             if (_tokens.length != _balances.length) revert MismatchedArrayLengths();
146             if (mintAmount == 0) revert InvalidTokenAmount();
161             if (redeemAmount == 0) revert InvalidTokenAmount();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


23              if (address(roleManagerAdmin) == address(0x0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


21              if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
22              if (address(_ezETHToken) == address(0x0)) revert InvalidZeroInput();
37              if (totalSupply == 0 || totalTVL == 0) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


72              if (!roleManager.isRestakeManagerAdmin(msg.sender)) revert NotRestakeManagerAdmin();
78              if (!roleManager.isDepositWithdrawPauser(msg.sender)) revert NotDepositWithdrawPauser();
84              if (msg.sender != address(depositQueue)) revert NotDepositQueue();
90              if (paused) revert ContractPaused();
146             if (_allocationBasisPoints > (100 * BASIS_POINTS)) revert OverMaxBasisPoints();
191             if (address(_operatorDelegator) == address(0x0)) revert InvalidZeroInput();
192             if (_allocationBasisPoints > (100 * BASIS_POINTS)) revert OverMaxBasisPoints();
206             if (!foundOd) revert NotFound();
224                 if (address(collateralTokens[i]) == address(_newCollateralToken)) revert AlreadyAdded();
367             if (operatorDelegators.length == 0) revert NotFound();
639             if (!found) revert NotFound();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


19              if (!roleManager.isNativeEthRestakeAdmin(msg.sender)) revert NotNativeEthRestakeAdmin();
25              if (!roleManager.isRestakeManagerAdmin(msg.sender)) revert NotRestakeManagerAdmin();
41              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
42              if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();
69              if (!success) revert TransferFailed();
75              if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


40              if (!roleManager.isWithdrawQueueAdmin(msg.sender)) revert NotWithdrawQueueAdmin();
46              if (msg.sender != address(restakeManager)) revert NotRestakeManager();
51              if (msg.sender != address(restakeManager.depositQueue())) revert NotDepositQueue();
109             if (_newBufferTarget.length == 0) revert InvalidZeroInput();
130             if (_newCoolDownPeriod == 0) revert InvalidZeroInput();
196             if (_asset == address(0) || _amount == 0) revert InvalidZeroInput();
208             if (_amount == 0 || _assetOut == address(0)) revert InvalidZeroInput();
211             if (withdrawalBufferTarget[_assetOut] == 0) revert UnsupportedWithdrawAsset();
236             if (amountToRedeem > getAvailableToWithdraw(_assetOut)) revert NotEnoughWithdrawBuffer();
287             if (block.timestamp - _withdrawRequest.createdAt < coolDownPeriod) revert EarlyClaim();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


16              if (!roleManager.isEzETHMinterBurner(msg.sender)) revert NotEzETHMinterBurner();
22              if (!roleManager.isTokenAdmin(msg.sender)) revert NotTokenAdmin();
34              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## NC092 - A event should be emitted if a non immutable state variable is set in a constructor:

  


```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


43          constructor(
44              address _router,
45              address _xRenzoBridgeL1,
46              uint64 _ccipEthChainSelector
47          ) CCIPReceiver(_router) {
48              if (_xRenzoBridgeL1 == address(0) || _ccipEthChainSelector == 0) revert InvalidZeroInput();
49      
50              // Set xRenzoBridge L1 contract address
51              xRenzoBridgeL1 = _xRenzoBridgeL1;
52      
53              // Set ccip source chain selector for Ethereum L1
54              ccipEthChainSelector = _ccipEthChainSelector;
55      
56              // Pause The contract to setup xRenzoDeposit
57              _pause();
58          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


52          constructor(address _connext, address _xRenzoBridgeL1, uint32 _connextEthChainDomain) {
53              if (_xRenzoBridgeL1 == address(0) || _connextEthChainDomain == 0 || _connext == address(0))
54                  revert InvalidZeroInput();
55      
56              // Set connext bridge address
57              connext = _connext;
58      
59              // Set xRenzoBridge L1 contract address
60              xRenzoBridgeL1 = _xRenzoBridgeL1;
61      
62              // Set connext source chain Domain Id for Ethereum L1
63              connextEthChainDomain = _connextEthChainDomain;
64      
65              // Pause The contract to setup xRenzoDeposit
66              _pause();
67          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

## NC093 - Unused file:

The file is never imported by any other source file. If the file is needed for tests, it should be moved to a test directory


```solidity
File: contracts/Bridge/Connext/libraries/LibConnextStorage.sol


/// @auditbase `contest/contracts/Bridge/Connext/libraries/LibConnextStorage.sol` not used in any contracts


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/libraries/LibConnextStorage.sol#L1:1

```solidity
File: contracts/Bridge/Connext/libraries/TokenId.sol


/// @auditbase `contest/contracts/Bridge/Connext/libraries/TokenId.sol` not used in any contracts


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/libraries/TokenId.sol#L1:1

## NC094 - Public state arrays should have a getter to return all elements:

In Solidity, public state variables automatically generate a getter function. For non-array types, this is straightforward: it simply returns the value. However, for arrays, the automatically generated getter only allows retrieval of an element at a specific index, not the entire array. This is mainly to prevent unintentional high gas costs, as returning the entire array can be expensive if it's large. If developers want to retrieve the whole array, they must explicitly define a function, as auto-generation could inadvertently expose contracts to gas-related vulnerabilities or lead to unwanted behavior for larger arrays.


```solidity
File: contracts/RestakeManagerStorage.sol


42          IOperatorDelegator[] public operatorDelegators;
49          IERC20[] public collateralTokens;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L0:0

## NC095 - It is best practice to use linear inheritance:

In Solidity, complex inheritance structures can obfuscate code understanding, introducing potential security risks. Multiple inheritance, especially with overlapping function names or state variables, can cause unintentional overrides or ambiguous behavior. Resolution: Strive for linear and simple inheritance chains. Avoid diamond or circular inheritance patterns. Clearly document the purpose and relationships of base contracts, ensuring that overrides are intentional. Tools like Remix or Hardhat can visualize inheritance chains, assisting in verification. Keeping inheritance streamlined aids in better code readability, reduces potential errors, and ensures smoother audits and upgrades.


<details>
<summary>Click to show 22 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


16      contract xRenzoBridge is
17          IXReceiver,
18          Initializable,
19          ReentrancyGuardUpgradeable,
20          xRenzoBridgeStorageV1


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


11      contract RenzoOracleL2 is Initializable, OwnableUpgradeable, RenzoOracleL2StorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


14      contract Receiver is CCIPReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


10      contract ConnextReceiver is IXReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


27      contract xRenzoDeposit is
28          Initializable,
29          OwnableUpgradeable,
30          ReentrancyGuardUpgradeable,
31          IRateProvider,
32          xRenzoDepositStorageV3


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


16      contract XERC20 is
17          Initializable,
18          ERC20Upgradeable,
19          OwnableUpgradeable,
20          IXERC20,
21          ERC20PermitUpgradeable


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


14      contract XERC20Factory is Initializable, IXERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


11      contract XERC20Lockbox is Initializable, IXERC20Lockbox {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


11      contract OptimismMintableXERC20 is ERC165Upgradeable, XERC20, IOptimismMintableERC20 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


14      contract OptimismMintableXERC20Factory is Initializable, XERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


17      contract OperatorDelegator is
18          Initializable,
19          ReentrancyGuardUpgradeable,
20          OperatorDelegatorStorageV4


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


10      contract DepositQueue is Initializable, ReentrancyGuardUpgradeable, DepositQueueStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


15      contract WBETHShim is Initializable, WBETHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


15      contract METHShim is Initializable, METHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


13      contract RenzoOracle is
14          IRenzoOracle,
15          Initializable,
16          ReentrancyGuardUpgradeable,
17          RenzoOracleStorageV1


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


14      contract RoleManager is IRoleManager, AccessControlUpgradeable, RoleManagerStorageV3 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


9       contract BalancerRateProvider is Initializable, IRateProvider, BalancerRateProviderStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


26      contract RestakeManager is Initializable, ReentrancyGuardUpgradeable, RestakeManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


16      contract RewardHandler is Initializable, ReentrancyGuardUpgradeable, RewardHandlerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


25      contract TimelockController is AccessControl, IERC721Receiver, IERC1155Receiver {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


11      contract WithdrawQueue is
12          Initializable,
13          PausableUpgradeable,
14          ReentrancyGuardUpgradeable,
15          WithdrawQueueStorageV1


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


13      contract EzEthToken is Initializable, ERC20Upgradeable, IEzEthToken, EzEthTokenStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## NC096 - Contracts with only unimplemented functions can be labeled as abstract:

A contract that's not meant to be deployed on its own but is intended to be inherited by other contracts should be marked as abstract. If a contract does not have any functions, mark it as abstract.  This ensures that developers recognize the contract's incomplete or intended-to-be-extended nature.


```solidity
File: contracts/Permissions/RoleManagerStorage.sol


9       contract RoleManagerStorageV1 {
37      contract RoleManagerStorageV2 is RoleManagerStorageV1 {
45      contract RoleManagerStorageV3 is RoleManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManagerStorage.sol#L0:0

```solidity
File: contracts/token/EzEthTokenStorage.sol


10      contract EzEthTokenStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthTokenStorage.sol#L0:0

## NC097 - Consider only defining one library/interface/contract per sol file:

Combining multiple libraries, interfaces, or contracts in a single  file can lead to clutter, reduced readability, and versioning issues. **Resolution**: Adopt the best practice of defining only one library, interface, or contract per Solidity file. This modular approach enhances clarity, simplifies unit testing, and streamlines code review. Furthermore, segregating components makes version management easier, as updates to one component won't necessitate changes to a file housing multiple unrelated components. Structured file management can further assist in avoiding naming collisions and ensure smoother integration into larger systems or DApps.


<details>
<summary>Click to show 6 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


9       interface IXERC20Registry {
10          function getXERC20(address erc20) external view returns (address xerc20);
11      
12          function getERC20(address xerc20) external view returns (address erc20);
13      
14          function getLockbox(address erc20) external view returns (address xerc20);
15      }
16      
17      interface L1StandardBridge {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


9       abstract contract xRenzoDepositStorageV1 is IxRenzoDeposit {
10          /// @notice The last timestamp the price was updated
11          uint256 public lastPriceTimestamp;
12      
13          /// @notice The last price that was updated - denominated in ETH with 18 decimal precision
14          uint256 public lastPrice;
15      
16          /// @notice The xezETH token address
17          IERC20 public xezETH;
18      
19          /// @notice The deposit token address - this is what users will deposit to mint xezETH
20          IERC20 public depositToken;
21      
22          /// @notice The collateral token address - this is what the deposit token will be swapped into and bridged to L1
23          IERC20 public collateralToken;
24      
25          /// @notice The address of the main Connext contract
26          IConnext public connext;
27      
28          /// @notice The swap ID for the connext token swap
29          bytes32 public swapKey;
30      
31          /// @notice The receiver middleware contract address
32          address public receiver;
33      
34          /// @notice The bridge router fee basis points - 100 basis points = 1%
35          uint256 public bridgeRouterFeeBps;
36      
37          /// @notice The bridge destination domain - mainnet ETH connext domain
38          uint32 public bridgeDestinationDomain;
39      
40          /// @notice The contract address where the bridge call should be sent on mainnet ETH
41          address public bridgeTargetAddress;
42      
43          /// @notice The mapping of allowed addresses that can trigger the bridge function
44          mapping(address => bool) public allowedBridgeSweepers;
45      }
46      
47      abstract contract xRenzoDepositStorageV2 is xRenzoDepositStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegatorStorage.sol


16      abstract contract OperatorDelegatorStorageV1 is IOperatorDelegator {
17          /// @dev reference to the RoleManager contract
18          IRoleManager public roleManager;
19      
20          /// @dev The main strategy manager contract in EigenLayer
21          IStrategyManager public strategyManager;
22      
23          /// @dev the restake manager contract
24          IRestakeManager public restakeManager;
25      
26          /// @dev The mapping of supported token addresses to their respective strategy addresses
27          /// This will control which tokens are supported by the protocol
28          mapping(IERC20 => IStrategy) public tokenStrategyMapping;
29      
30          /// @dev The address to delegate tokens to in EigenLayer
31          address public delegateAddress;
32      
33          /// @dev the delegation manager contract
34          IDelegationManager public delegationManager;
35      
36          /// @dev the EigenLayer EigenPodManager contract
37          IEigenPodManager public eigenPodManager;
38      
39          /// @dev The EigenPod owned by this contract
40          IEigenPod public eigenPod;
41      
42          /// @dev Tracks the balance that was staked to validators but hasn't been restaked to EL yet
43          uint256 public stakedButNotVerifiedEth;
44      }
45      
46      abstract contract OperatorDelegatorStorageV2 is OperatorDelegatorStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegatorStorage.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueueStorage.sol


9       abstract contract DepositQueueStorageV1 is IDepositQueue {
10          /// @dev reference to the RoleManager contract
11          IRoleManager public roleManager;
12      
13          /// @dev the address of the RestakeManager contract
14          IRestakeManager public restakeManager;
15      
16          /// @dev the address where fees will be sent - must be non zero to enable fees
17          address public feeAddress;
18      
19          /// @dev the basis points to charge for fees - 100 basis points = 1%
20          uint256 public feeBasisPoints;
21      
22          /// @dev the total amount the protocol has earned - token address => amount
23          mapping(address => uint256) public totalEarned;
24      }
25      
26      abstract contract DepositQueueStorageV2 is DepositQueueStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueueStorage.sol#L0:0

```solidity
File: contracts/Permissions/RoleManagerStorage.sol


9       contract RoleManagerStorageV1 {
10          /// @dev role for granting capability to mint/burn ezETH
11          bytes32 public constant RX_ETH_MINTER_BURNER = keccak256("RX_ETH_MINTER_BURNER");
12      
13          /// @dev role for granting capability to update config on the OperatorDelgator Contracts
14          bytes32 public constant OPERATOR_DELEGATOR_ADMIN = keccak256("OPERATOR_DELEGATOR_ADMIN");
15      
16          /// @dev role for granting capability to update config on the Oracle Contract
17          bytes32 public constant ORACLE_ADMIN = keccak256("ORACLE_ADMIN");
18      
19          /// @dev role for granting capability to update config on the Restake Manager
20          bytes32 public constant RESTAKE_MANAGER_ADMIN = keccak256("RESTAKE_MANAGER_ADMIN");
21      
22          /// @dev role for granting capability to update config on the Token Contract
23          bytes32 public constant TOKEN_ADMIN = keccak256("TOKEN_ADMIN");
24      
25          /// @dev role for granting capability to restake native ETH
26          bytes32 public constant NATIVE_ETH_RESTAKE_ADMIN = keccak256("NATIVE_ETH_RESTAKE_ADMIN");
27      
28          /// @dev role for sweeping ERC20 Rewards
29          bytes32 public constant ERC20_REWARD_ADMIN = keccak256("ERC20_REWARD_ADMIN");
30      
31          /// @dev role for pausing deposits and withdraws on RestakeManager
32          bytes32 public constant DEPOSIT_WITHDRAW_PAUSER = keccak256("DEPOSIT_WITHDRAW_PAUSER");
33      }
34      
35      /// On the next version of the protocol, if new variables are added, put them in the below
36      /// contract and use this as the inheritance chain.
37      contract RoleManagerStorageV2 is RoleManagerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManagerStorage.sol#L0:0

```solidity
File: contracts/RestakeManagerStorage.sol


15      abstract contract RestakeManagerStorageV1 is IRestakeManager {
16          /// @dev reference to the RoleManager contract
17          IRoleManager public roleManager;
18      
19          /// @dev reference to the ezETH token contract
20          IEzEthToken public ezETH;
21      
22          /// @dev reference to the strategyManager contract in EigenLayer
23          IStrategyManager public strategyManager;
24      
25          /// @dev reference to the delegationManager contract in EigenLayer
26          IDelegationManager public delegationManager;
27      
28          /// @dev data stored for a withdrawal
29          struct PendingWithdrawal {
30              uint256 ezETHToBurn;
31              address withdrawer;
32              IERC20 tokenToWithdraw;
33              uint256 tokenAmountToWithdraw;
34              IOperatorDelegator operatorDelegator;
35              bool completed;
36          }
37      
38          /// @dev mapping of pending withdrawals, indexed by the withdrawal root from EigenLayer
39          mapping(bytes32 => PendingWithdrawal) public pendingWithdrawals;
40      
41          /// @dev Stores the list of OperatorDelegators
42          IOperatorDelegator[] public operatorDelegators;
43      
44          /// @dev Mapping to store the allocations to each operatorDelegator
45          /// Stored in basis points (e.g. 1% = 100)
46          mapping(IOperatorDelegator => uint256) public operatorDelegatorAllocations;
47      
48          /// @dev Stores the list of collateral tokens
49          IERC20[] public collateralTokens;
50      
51          /// @dev Reference to the oracle contract
52          IRenzoOracle public renzoOracle;
53      
54          /// @dev Controls pause state of contract
55          bool public paused;
56      
57          /// @dev The max amount of TVL allowed.  If this is set to 0, no max TVL is enforced
58          uint256 public maxDepositTVL;
59      
60          /// @dev Reference to the deposit queue contract
61          IDepositQueue public depositQueue;
62      }
63      
64      abstract contract RestakeManagerStorageV2 is RestakeManagerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L0:0

</details>









## G001 - Don't Initialize Variables with Default Value:

  


<details>
<summary>Click to show 40 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


for (uint256 i = 0; i < _destinationParam.length; ) {

for (uint256 i = 0; i < _connextDestinationParam.length; ) {

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L264:264

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


uint256 minOut = 0;

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L372:372

```solidity
File: contracts/Delegation/OperatorDelegator.sol


for (uint256 i = 0; i < strategyLength; i++) {

for (uint256 i = 0; i < validatorFields.length; ) {

uint256 gasRefunded = 0;

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L507:507

```solidity
File: contracts/Deposits/DepositQueue.sol


uint256 feeAmount = 0;

for (uint256 i = 0; i < arrayLength; ) {

uint256 feeAmount = 0;

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L257:257

```solidity
File: contracts/Oracle/RenzoOracle.sol


uint256 totalValue = 0;

for (uint256 i = 0; i < tokenLength; ) {

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L111:111

```solidity
File: contracts/RestakeManager.sol


for (uint256 i = 0; i < odLength; ) {

for (uint256 i = 0; i < odLength; ) {

bool foundOd = false;

for (uint256 i = 0; i < odLength; ) {

for (uint256 i = 0; i < tokenLength; ) {

for (uint256 i = 0; i < tokenLength; ) {

uint256 totalTVL = 0;

bool withdrawQueueTokenBalanceRecorded = false;

uint256 totalWithdrawalQueueValue = 0;

for (uint256 i = 0; i < odLength; ) {

uint256 operatorTVL = 0;

for (uint256 j = 0; j < tokenLength; ) {

for (uint256 i = 0; i < tvlLength; ) {

for (uint256 i = 0; i < odLength; ) {

for (uint256 i = 0; i < odLength; ) {

for (uint256 i = 0; i < tokenLength; ) {

uint256 currentTokenTVL = 0;

for (uint256 i = 0; i < odLength; ) {

bool found = false;

for (uint256 i = 0; i < odLength; ) {

uint256 totalRewards = 0;

for (uint256 i = 0; i < tokenLength; ) {

for (uint256 i = 0; i < odLength; ) {

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L699:699

```solidity
File: contracts/TimelockController.sol


for (uint256 i = 0; i < proposers.length; ++i) {

for (uint256 i = 0; i < executors.length; ++i) {

for (uint256 i = 0; i < targets.length; ++i) {

for (uint256 i = 0; i < targets.length; ++i) {

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L355:355

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


for (uint256 i = 0; i < _withdrawalBufferTarget.length; ) {

for (uint256 i = 0; i < _newBufferTarget.length; ) {

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L110:110

</details>

## G002 - Long Revert Strings:

  


<details>
<summary>Click to show 11 findings</summary>

```solidity
File: contracts/TimelockController.sol


require(targets.length == values.length, "TimelockController: length mismatch");

require(targets.length == payloads.length, "TimelockController: length mismatch");

require(!isOperation(id), "TimelockController: operation already scheduled");

require(delay >= getMinDelay(), "TimelockController: insufficient delay");

require(isOperationPending(id), "TimelockController: operation cannot be cancelled");

require(targets.length == values.length, "TimelockController: length mismatch");

require(targets.length == payloads.length, "TimelockController: length mismatch");

require(success, "TimelockController: underlying transaction reverted");

require(isOperationReady(id), "TimelockController: operation is not ready");

require(isOperationReady(id), "TimelockController: operation is not ready");

require(msg.sender == address(this), "TimelockController: caller must be timelock");

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L403:403

</details>

## G003 - Use assembly to check for `address(0)`:

*Saves 6 gas per instance*


<details>
<summary>Click to show 12 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


41              if (_blastStandardBridge == address(0) || _registry == address(0)) {
41              if (_blastStandardBridge == address(0) || _registry == address(0)) {
73              if (xerc20 == address(0) || lockbox == address(0)) {
73              if (xerc20 == address(0) || lockbox == address(0)) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


84                  address(_ezETH) == address(0) ||
85                  address(_xezETH) == address(0) ||
86                  address(_restakeManager) == address(0) ||
87                  address(_wETH) == address(0) ||
88                  address(_xezETHLockbox) == address(0) ||
89                  address(_connext) == address(0) ||
90                  address(_linkRouterClient) == address(0) ||
91                  address(_rateProvider) == address(0) ||
92                  address(_linkToken) == address(0) ||
93                  address(_roleManager) == address(0)


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


27              if (address(_oracle) == address(0)) revert InvalidZeroInput();
37              if (address(_oracleAddress) == address(0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


48              if (_xRenzoBridgeL1 == address(0) || _ccipEthChainSelector == 0) revert InvalidZeroInput();
97              if (_newXRenzoBridgeL1 == address(0)) revert InvalidZeroInput();
135             if (_newXRenzoDeposit == address(0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


53              if (_xRenzoBridgeL1 == address(0) || _connextEthChainDomain == 0 || _connext == address(0))
53              if (_xRenzoBridgeL1 == address(0) || _connextEthChainDomain == 0 || _connext == address(0))
93              if (_newXRenzoBridgeL1 == address(0)) revert InvalidZeroInput();
131             if (_newXRenzoDeposit == address(0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


93                  address(_xezETH) == address(0) ||
94                  address(_depositToken) == address(0) ||
95                  address(_collateralToken) == address(0) ||
96                  address(_connext) == address(0) ||
99                  _bridgeTargetAddress == address(0)
291             if (receiver == address(0) && address(oracle) == address(0)) revert PriceFeedNotAvailable();
291             if (receiver == address(0) && address(oracle) == address(0)) revert PriceFeedNotAvailable();
292             if (address(oracle) != address(0)) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


111             if ((_baseToken == address(0) && !_isNative) || (_isNative && _baseToken != address(0))) {
111             if ((_baseToken == address(0) && !_isNative) || (_isNative && _baseToken != address(0))) {
116             if (_lockboxRegistry[_xerc20] != address(0)) revert IXERC20Factory_LockboxAlreadyDeployed();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


217                     if (address(tokenStrategyMapping[tokens[i]]) == address(0))
278                 if (address(tokens[i]) == address(0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


88              if (address(_withdrawQueue) == address(0)) revert InvalidZeroInput();
138             if (_amount == 0 || _asset == address(0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/TimelockController.sol


102             if (admin != address(0)) {
245             if (salt != bytes32(0)) {
275             if (salt != bytes32(0)) {
379                 predecessor == bytes32(0) || isOperationDone(predecessor),


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


73                  address(_roleManager) == address(0) ||
74                  address(_ezETH) == address(0) ||
75                  address(_renzoOracle) == address(0) ||
76                  address(_restakeManager) == address(0) ||
90                      _withdrawalBufferTarget[i].asset == address(0) ||
111                 if (_newBufferTarget[i].asset == address(0) || _newBufferTarget[i].bufferAmount == 0)
196             if (_asset == address(0) || _amount == 0) revert InvalidZeroInput();
208             if (_amount == 0 || _assetOut == address(0)) revert InvalidZeroInput();


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


69              if (from != address(0) && to != address(0)) {
69              if (from != address(0) && to != address(0)) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## G004 - `internal` functions not called by the contract should be removed to save deployment gas:

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword


```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


    function _ccipReceive(
        Client.Any2EVMMessage memory any2EvmMessage
    ) internal override whenNotPaused {
        address _ccipSender = abi.decode(any2EvmMessage.sender, (address));
        uint64 _ccipSourceChainSelector = any2EvmMessage.sourceChainSelector;
        // Verify origin on the price feed
        if (_ccipSender != xRenzoBridgeL1) revert InvalidSender(xRenzoBridgeL1, _ccipSender);
        // Verify Source chain of the message
        if (_ccipSourceChainSelector != ccipEthChainSelector)
            revert InvalidSourceChain(ccipEthChainSelector, _ccipSourceChainSelector);
        (uint256 _price, uint256 _timestamp) = abi.decode(any2EvmMessage.data, (uint256, uint256));
        xRenzoDeposit.updatePrice(_price, _timestamp);
        emit MessageReceived(
            any2EvmMessage.messageId,
            _ccipSourceChainSelector,
            _ccipSender,
            _price,
            _timestamp
        );
    }

```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L66:85


## G005 - Multiple address/ID mappings can be combined into a single mapping of an address/ID to a struct, where appropriate:

Saves a storage slot for the mapping. Depending on the circumstances and sizes of types, can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they both fit in the same storage slot. Finally, if both fields are accessed in the same function, can save ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations.


```solidity
File: contracts/Delegation/OperatorDelegatorStorage.sol


56          mapping(address => uint256) public adminGasSpentInWei;
57      }
58      
59      abstract contract OperatorDelegatorStorageV4 is OperatorDelegatorStorageV3 {
60          /// @dev mapping of token shares in withdraw queue of EigenLayer
61          mapping(address => uint256) public queuedShares;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegatorStorage.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueueStorage.sol


44          mapping(address => uint256) public withdrawalBufferTarget;
45      
46          /// @dev mapping of claimReserve (already withdraw requested), indexed by token address
47          mapping(address => uint256) public claimReserve;
48      
49          /// @dev mapiing of withdraw requests array, indexed by user address
50          mapping(address => WithdrawRequest[]) public withdrawRequests;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueueStorage.sol#L0:0

## G006 - Using storage instead of `memory` for structs/arrays saves gas:

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct


```solidity
File: contracts/Withdraw/WithdrawQueue.sol


284             WithdrawRequest memory _withdrawRequest = withdrawRequests[msg.sender][
285                 withdrawRequestIndex
286             ];


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

## G007 - Multiple accesses of a mapping/array should use a local variable cache.:

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata


<details>
<summary>Click to show 7 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


253                     _destinationParam[i]._renzoReceiver,
279                     _connextDestinationParam[i].relayerFee


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


179                 bridges[_bridge].minterParams.ratePerSecond
195                 bridges[_bridge].burnerParams.ratePerSecond
208             bridges[_bridge].minterParams.currentLimit = _currentLimit - _change;
220             bridges[_bridge].burnerParams.currentLimit = _currentLimit - _change;
242             bridges[_bridge].minterParams.timestamp = block.timestamp;
264             bridges[_bridge].burnerParams.timestamp = block.timestamp;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


249                 queuedWithdrawalParams[0].shares
232                 queuedShares[address(tokens[i])] += queuedWithdrawalParams[0].shares[i];
232                 queuedShares[address(tokens[i])] += queuedWithdrawalParams[0].shares[i];
225                         .underlyingToSharesView(tokenAmounts[i]);
224                     queuedWithdrawalParams[0].shares[i] = tokenStrategyMapping[tokens[i]]
308                         _deposit(tokens[i], balanceOfToken);
333                             queuedShares[address(token)]
332                         tokenStrategyMapping[token].sharesToUnderlyingView(
344                     : queuedShares[IS_NATIVE] + stakedButNotVerifiedEth + uint256(podOwnerShares);
489             adminGasSpentInWei[tx.origin] -= gasRefund;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


179             totalEarned[address(0x0)] = totalEarned[address(0x0)] + remainingRewards;
237                     operatorDelegators[i],
238                     pubkeys[i],
272                 totalEarned[address(token)] = totalEarned[address(token)] + balance - feeAmount;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


172                     operatorDelegators[i] = operatorDelegators[operatorDelegators.length - 1];
251                     collateralTokens[i] = collateralTokens[collateralTokens.length - 1];
329                 uint256 operatorEthBalance = operatorDelegators[i].getStakedETHBalance();
319                             collateralTokens[j].balanceOf(withdrawQueue)
313                     operatorTVL += operatorValues[j];
392             return operatorDelegators[0];
383                     return operatorDelegators[i];
436                 if (operatorDelegatorTokenTVLs[i][tokenIndex] >= ezETHValue) {
437                     return operatorDelegators[i];
529                 if (currentTokenTVL + collateralTokenValue > collateralTokenTvlLimits[_collateralToken])
688                 totalRewards += renzoOracle.lookupTokenValue(collateralTokens[i], tokenRewardAmount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/TimelockController.sol


109                 _setupRole(CANCELLER_ROLE, proposers[i]);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


93                  withdrawalBufferTarget[_withdrawalBufferTarget[i].asset] = _withdrawalBufferTarget[i]
117                 withdrawalBufferTarget[_newBufferTarget[i].asset] = _newBufferTarget[i].bufferAmount;
117                 withdrawalBufferTarget[_newBufferTarget[i].asset] = _newBufferTarget[i].bufferAmount;
160                 return address(this).balance - claimReserve[_asset];
174                     ? withdrawalBufferTarget[_asset] - availableToWithdraw
261                 withdrawRequests[msg.sender].length - 1
296             withdrawRequests[msg.sender].pop();
293             withdrawRequests[msg.sender][withdrawRequestIndex] = withdrawRequests[msg.sender][


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## G008 - Internal functions only called once can be inlined to save gas:

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.


<details>
<summary>Click to show 7 findings</summary>

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


367         function _trade(uint256 _amountIn, uint256 _deadline) internal returns (uint256) {
396         function _recoverBridgeFee() internal {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


76          function __XERC20_init(
348         function _mintWithCaller(address _caller, address _user, uint256 _amount) internal {
328         function _burnWithCaller(address _caller, address _user, uint256 _amount) internal {
230         function _changeMinterLimit(address _bridge, uint256 _limit) internal {
252         function _changeBurnerLimit(address _bridge, uint256 _limit) internal {
217         function _useBurnerLimits(address _bridge, uint256 _change) internal {
205         function _useMinterLimits(address _bridge, uint256 _change) internal {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


135         function _deployXERC20(
184         function _deployLockbox(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


73          function _deployOptimismMintableXERC20(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


481         function _refundGas() internal returns (uint256) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


69          function _getWBETHData()


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


69          function _getMETHData()


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

</details>

## G009 - Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement:

`require(a <= b); x = b - a` => `require(a <= b); unchecked { x = b - a }`


```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


287                 _difference = _limit - _oldLimit;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

## G010 - Optimize names to save gas:

public/external function names and public member variable names can be optimized to save gas.


<details>
<summary>Click to show 24 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


9       interface IXERC20Registry {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


16      contract xRenzoBridge is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


11      contract RenzoOracleL2 is Initializable, OwnableUpgradeable, RenzoOracleL2StorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


14      contract Receiver is CCIPReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


10      contract ConnextReceiver is IXReceiver, Ownable, Pausable {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


27      contract xRenzoDeposit is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


16      contract XERC20 is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


14      contract XERC20Factory is Initializable, IXERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


11      contract XERC20Lockbox is Initializable, IXERC20Lockbox {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


11      contract OptimismMintableXERC20 is ERC165Upgradeable, XERC20, IOptimismMintableERC20 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


14      contract OptimismMintableXERC20Factory is Initializable, XERC20Factory {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


17      contract OperatorDelegator is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


10      contract DepositQueue is Initializable, ReentrancyGuardUpgradeable, DepositQueueStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/IRestakeManager.sol


7       interface IRestakeManager {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/IRestakeManager.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


15      contract WBETHShim is Initializable, WBETHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


15      contract METHShim is Initializable, METHShimStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


13      contract RenzoOracle is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


14      contract RoleManager is IRoleManager, AccessControlUpgradeable, RoleManagerStorageV3 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


9       contract BalancerRateProvider is Initializable, IRateProvider, BalancerRateProviderStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


26      contract RestakeManager is Initializable, ReentrancyGuardUpgradeable, RestakeManagerStorageV2 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


16      contract RewardHandler is Initializable, ReentrancyGuardUpgradeable, RewardHandlerStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


25      contract TimelockController is AccessControl, IERC721Receiver, IERC1155Receiver {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


11      contract WithdrawQueue is


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


13      contract EzEthToken is Initializable, ERC20Upgradeable, IEzEthToken, EzEthTokenStorageV1 {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## G011 - Structs should group like types together to save gas:

Structs can be more gas-efficient by grouping together members of the same type. This ordering can potentially save gas.


```solidity
File: contracts/Bridge/Connext/libraries/LibConnextStorage.sol


38      struct TransferInfo {
39          uint32 originDomain;
40          uint32 destinationDomain;
41          uint32 canonicalDomain;
42          address to;
43          address delegate;
44          bool receiveLocal;
45          bytes callData;
46          uint256 slippage;
47          address originSender;
48          uint256 bridgedAmt;
49          uint256 normalizedIn;
50          uint256 nonce;
51          bytes32 canonicalId;
52      }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/libraries/LibConnextStorage.sol#L0:0

```solidity
File: contracts/RestakeManagerStorage.sol


29          struct PendingWithdrawal {
30              uint256 ezETHToBurn;
31              address withdrawer;
32              IERC20 tokenToWithdraw;
33              uint256 tokenAmountToWithdraw;
34              IOperatorDelegator operatorDelegator;
35              bool completed;
36          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L0:0

## G012 - The result of function calls should be cached rather than re-calling the function:

Caching the result of a function call in a local variable when the function is called multiple times can save gas due to avoiding the need to execute the function code multiple times.


```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


30              if (_oracle.decimals() > 18) revert InvalidTokenDecimals(18, _oracle.decimals());
40                  revert InvalidTokenDecimals(18, _oracleAddress.decimals());


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


297                         tokens[i].safeApprove(address(restakeManager.depositQueue()), bufferToFill);
300                         restakeManager.depositQueue().fillERC20withdrawBuffer(
519                 address destination = address(restakeManager.depositQueue());


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


62                  revert InvalidTokenDecimals(18, _oracleAddress.decimals());


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

## G013 - Stack variable used as a cheaper cache for a state variable is only used once:

If the variable is only accessed once, it's cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.


```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


237                 _oldLimit,
259                 _oldLimit,


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

## G014 - Constructors can be marked payable:

Payable functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided. A constructor can safely be marked as payable, since only the deployer would be able to pass funds, and the project itself would not pass any funds.


<details>
<summary>Click to show 23 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


39          constructor(address _blastStandardBridge, address _registry) {
40              // Sanity check
41              if (_blastStandardBridge == address(0) || _registry == address(0)) {
42                  revert InvalidAddress();
43              }
44      
45              blastStandardBridge = _blastStandardBridge;
46              registry = _registry;
47          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


65          constructor() {
66              _disableInitializers();
67          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


19          constructor() {
20              _disableInitializers();
21          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


43          constructor(
44              address _router,
45              address _xRenzoBridgeL1,
46              uint64 _ccipEthChainSelector
47          ) CCIPReceiver(_router) {
48              if (_xRenzoBridgeL1 == address(0) || _ccipEthChainSelector == 0) revert InvalidZeroInput();
49      
50              // Set xRenzoBridge L1 contract address
51              xRenzoBridgeL1 = _xRenzoBridgeL1;
52      
53              // Set ccip source chain selector for Ethereum L1
54              ccipEthChainSelector = _ccipEthChainSelector;
55      
56              // Pause The contract to setup xRenzoDeposit
57              _pause();
58          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


52          constructor(address _connext, address _xRenzoBridgeL1, uint32 _connextEthChainDomain) {
53              if (_xRenzoBridgeL1 == address(0) || _connextEthChainDomain == 0 || _connext == address(0))
54                  revert InvalidZeroInput();
55      
56              // Set connext bridge address
57              connext = _connext;
58      
59              // Set xRenzoBridge L1 contract address
60              xRenzoBridgeL1 = _xRenzoBridgeL1;
61      
62              // Set connext source chain Domain Id for Ethereum L1
63              connextEthChainDomain = _connextEthChainDomain;
64      
65              // Pause The contract to setup xRenzoDeposit
66              _pause();
67          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


59          constructor() {
60              _disableInitializers();
61          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


50          constructor() {
51              _disableInitializers();
52          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


44          constructor() {
45              _disableInitializers();
46          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


33          constructor() {
34              _disableInitializers();
35          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


24          constructor() {
25              _disableInitializers();
26          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


19          constructor() {
20              _disableInitializers();
21          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


69          constructor() {
70              _disableInitializers();
71          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


69          constructor() {
70              _disableInitializers();
71          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


18          constructor() {
19              _disableInitializers();
20          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


18          constructor() {
19              _disableInitializers();
20          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


39          constructor() {
40              _disableInitializers();
41          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


17          constructor() {
18              _disableInitializers();
19          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


12          constructor() {
13              _disableInitializers();
14          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


96          constructor() {
97              _disableInitializers();
98          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


33          constructor() {
34              _disableInitializers();
35          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


87          constructor(
88              uint256 minDelay,
89              address[] memory proposers,
90              address[] memory executors,
91              address admin
92          ) {
93              _setRoleAdmin(TIMELOCK_ADMIN_ROLE, TIMELOCK_ADMIN_ROLE);
94              _setRoleAdmin(PROPOSER_ROLE, TIMELOCK_ADMIN_ROLE);
95              _setRoleAdmin(EXECUTOR_ROLE, TIMELOCK_ADMIN_ROLE);
96              _setRoleAdmin(CANCELLER_ROLE, TIMELOCK_ADMIN_ROLE);
97      
98              // self administration
99              _setupRole(TIMELOCK_ADMIN_ROLE, address(this));
100     
101             // optional admin
102             if (admin != address(0)) {
103                 _setupRole(TIMELOCK_ADMIN_ROLE, admin);
104             }
105     
106             // register proposers and cancellers
107             for (uint256 i = 0; i < proposers.length; ++i) {
108                 _setupRole(PROPOSER_ROLE, proposers[i]);
109                 _setupRole(CANCELLER_ROLE, proposers[i]);
110             }
111     
112             // register executors
113             for (uint256 i = 0; i < executors.length; ++i) {
114                 _setupRole(EXECUTOR_ROLE, executors[i]);
115             }
116     
117             _minDelay = minDelay;
118             emit MinDelayChange(0, minDelay);
119         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


57          constructor() {
58              _disableInitializers();
59          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


28          constructor() {
29              _disableInitializers();
30          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## G015 - Use solidity version 0.8.20 or above to improve gas performance:

Upgrade to the latest solidity version 0.8.20 to get additional gas savings. See the latest release for reference: https://blog.soliditylang.org/2023/05/10/solidity-0.8.20-release-announcement/


<details>
<summary>Click to show 5 findings</summary>

```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


2       pragma solidity ^0.8.13;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/Connext/libraries/LibConnextStorage.sol


2       pragma solidity ^0.8.0;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/libraries/LibConnextStorage.sol#L0:0

```solidity
File: contracts/Bridge/Connext/libraries/TokenId.sol


2       pragma solidity ^0.8.0;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/libraries/TokenId.sol#L0:0

```solidity
File: contracts/TimelockController.sol


4       pragma solidity ^0.8.0;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


2       pragma solidity ^0.8.9;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## G016 - Expression `` is cheaper than `new bytes(0)`:

  


```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


199             bytes memory returnData = new bytes(0);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

## G017 - Use assembly to emit events:

Using the [scratch space](https://github.com/Vectorized/solady/blob/30558f5402f02351b96eeb6eaf32bcea29773841/src/tokens/ERC1155.sol#L501-L504) for event arguments (two words or fewer) will save gas over needing Solidity's full abi memory expansion used for emitting normally. 


<details>
<summary>Click to show 16 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


196             emit EzETHMinted(_transferId, _amount, _origin, _originSender, ezETHAmount);
250                 emit MessageSent(
251                     messageId,
252                     _destinationParam[i].destinationChainSelector,
253                     _destinationParam[i]._renzoReceiver,
254                     exchangeRate,
255                     address(linkToken),
256                     fees
257                 );
275                 emit ConnextMessageSent(
276                     _connextDestinationParam[i].destinationDomainId,
277                     _connextDestinationParam[i]._renzoReceiver,
278                     exchangeRate,
279                     _connextDestinationParam[i].relayerFee
280                 );


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


42              emit OracleAddressUpdated(address(_oracleAddress), address(oracle));


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


78              emit MessageReceived(
79                  any2EvmMessage.messageId,
80                  _ccipSourceChainSelector,
81                  _ccipSender,
82                  _price,
83                  _timestamp
84              );
98              emit XRenzoBridgeL1Updated(_newXRenzoBridgeL1, xRenzoBridgeL1);
109             emit CCIPEthChainSelectorUpdated(_newChainSelector, ccipEthChainSelector);
136             emit XRenzoDepositUpdated(_newXRenzoDeposit, address(xRenzoDeposit));


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


80              emit MessageReceived(_transferId, _origin, _originSender, _price, _timestamp);
94              emit XRenzoBridgeL1Updated(_newXRenzoBridgeL1, xRenzoBridgeL1);
105             emit ConnextEthChainDomainUpdated(_newChainDomain, connextEthChainDomain);
132             emit XRenzoDepositUpdated(_newXRenzoDeposit, address(xRenzoDeposit));


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


269             emit Deposit(msg.sender, _amountIn, xezETHAmount);
358             emit PriceUpdated(_price, _timestamp);
405             emit SweeperBridgeFeeCollected(msg.sender, feeCollected);
448             emit BridgeSwept(bridgeDestinationDomain, bridgeTargetAddress, msg.sender, balance);
469             emit BridgeSweeperAddressUpdated(_sweeper, _allowed);
502             emit OraclePriceFeedUpdated(address(_oracle), address(oracle));
512             emit ReceiverPriceFeedUpdated(_receiver, receiver);
523             emit BridgeFeeShareUpdated(bridgeFeeShare, _newShare);
534             emit SweepBatchSizeUpdated(sweepBatchSize, _newBatchSize);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


125             emit LockboxSet(_lockbox);
142             emit BridgeLimitsSet(_mintingLimit, _burningLimit, _bridge);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


91              emit XERC20Deployed(_xerc20);
120             emit LockboxDeployed(_lockbox);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


126             emit Withdraw(_to, _amount);
151             emit Deposit(_to, _amount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


56              emit XERC20Deployed(_xerc20);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


113             emit TokenStrategyUpdated(_token, _strategy);
129             emit DelegationAddressUpdated(_delegateAddress);
136             emit BaseGasAmountSpentUpdated(baseGasAmountSpent, _baseGasAmountSpent);
241             emit WithdrawStarted(
242                 withdrawalRoot,
243                 address(this),
244                 delegateAddress,
245                 address(this),
246                 nonce,
247                 block.number,
248                 queuedWithdrawalParams[0].strategies,
249                 queuedWithdrawalParams[0].shares
250             );
317             emit WithdrawCompleted(
318                 delegationManager.calculateWithdrawalRoot(withdrawal),
319                 withdrawal.strategies,
320                 withdrawal.shares
321             );
473             emit GasSpent(msg.sender, gasSpent);
491             emit GasRefunded(tx.origin, gasRefund);
523                 emit RewardsForwarded(destination, remainingAmount);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


89              emit WithdrawQueueUpdated(address(withdrawQueue), address(_withdrawQueue));
108             emit FeeConfigUpdated(_feeAddress, _feeBasisPoints);
117             emit RestakeManagerUpdated(_restakeManager);
125             emit ETHDepositedFromProtocol(msg.value);
155             emit FullWithdrawalETHReceived(msg.value);
171                 emit ProtocolFeesPaid(IERC20(address(0x0)), feeAmount, feeAddress);
182             emit RewardsDeposited(IERC20(address(0x0)), remainingRewards);
202             emit ETHStakedFromQueue(operatorDelegator, pubkey, 32 ether, address(this).balance);
236                 emit ETHStakedFromQueue(
237                     operatorDelegators[i],
238                     pubkeys[i],
239                     32 ether,
240                     address(this).balance
241                 );
264                     emit ProtocolFeesPaid(token, feeAmount, feeAddress);
275                 emit RewardsDeposited(IERC20(address(token)), balance - feeAmount);
288             emit GasRefunded(msg.sender, gasRefund);
304                 emit BufferFilled(IS_NATIVE, bufferToFill);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


65              emit OracleAddressUpdated(_token, _oracleAddress);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


151             emit OperatorDelegatorAdded(_newOperatorDelegator);
156             emit OperatorDelegatorAllocationUpdated(_newOperatorDelegator, _allocationBasisPoints);
169                     emit OperatorDelegatorAllocationUpdated(_operatorDelegatorToRemove, 0);
174                     emit OperatorDelegatorRemoved(_operatorDelegatorToRemove);
211             emit OperatorDelegatorAllocationUpdated(_operatorDelegator, _allocationBasisPoints);
240             emit CollateralTokenAdded(_newCollateralToken);
253                     emit CollateralTokenRemoved(_collateralTokenToRemove);
575             emit Deposit(msg.sender, _collateralToken, _amount, ezETHToMint, _referralId);
615             emit Deposit(msg.sender, IERC20(address(0x0)), msg.value, ezETHToMint, _referralId);
716             emit CollateralTokenTvlUpdated(_token, _limit);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


47              emit RewardDestinationUpdated(_rewardDestination);
79              emit RewardDestinationUpdated(_rewardDestination);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


118             emit MinDelayChange(0, minDelay);
244             emit CallScheduled(id, 0, target, value, data, predecessor, delay);
246                 emit CallSalt(id, salt);
273                 emit CallScheduled(id, i, targets[i], values[i], payloads[i], predecessor, delay);
276                 emit CallSalt(id, salt);
300             emit Cancelled(id);
326             emit CallExecuted(id, 0, target, value, payload);
360                 emit CallExecuted(id, i, target, value, payload);
404             emit MinDelayChange(_minDelay, newDelay);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


113                 emit WithdrawBufferTargetUpdated(
114                     withdrawalBufferTarget[_newBufferTarget[i].asset],
115                     _newBufferTarget[i].bufferAmount
116                 );
131             emit CoolDownPeriodUpdated(coolDownPeriod, _newCoolDownPeriod);
183             emit EthBufferFilled(msg.value);
198             emit ERC20BufferFilled(_asset, _amount);
255             emit WithdrawRequestCreated(
256                 withdrawRequestNonce,
257                 msg.sender,
258                 _assetOut,
259                 amountToRedeem,
260                 _amount,
261                 withdrawRequests[msg.sender].length - 1
262             );
311             emit WithdrawRequestClaimed(_withdrawRequest);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## G018 - Fewer storage slots can be used by storing timestamps in types smaller than uint256:

Ethereum's block.timestamp can be stored in a type smaller than uint256.   A uint32 variable can store a timestamp until year 2106.


```solidity
File: contracts/Bridge/L2/xRenzoDepositStorage.sol


11          uint256 public lastPriceTimestamp;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDepositStorage.sol#L0:0


## G019 - Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead:

> When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.htmlEach operation involving a `uint8` costs an extra [**22-28 gas**](https://gist.github.com/IllIllI000/9388d20c70f9a4632eb3ca7836f54977) (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.


```solidity
File: contracts/Delegation/OperatorDelegator.sol


382                 uint64 validatorCurrentBalanceGwei = BeaconChainProofs.getEffectiveBalanceGwei(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

## G020 - Consider activating `via-ir` for deploying:


        The IR-based code generator was introduced with an aim to not only allow code generation to be more transparent and auditable but also to enable more powerful optimization passes that span across functions.

        You can enable it on the command line using `--via-ir` or with the option `{"viaIR": true}`.

        This will take longer to compile, but you can just simple test it before deploying and if you got a better benchmark then you can add --via-ir to your deploy command

        More on: https://docs.soliditylang.org/en/v0.8.17/ir-breaking-changes.html
        


```solidity
File: Various Files


None

```

## G021 - Emit Used In Loop:

Emitting an event inside a loop performs a LOG op N times, where N is the loop length. Consider refactoring the code to emit the event only once at the end of loop. Gas savings should be multiplied by the average loop length.


<details>
<summary>Click to show 5 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


250                 emit MessageSent(
275                 emit ConnextMessageSent(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


236                 emit ETHStakedFromQueue(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


169                     emit OperatorDelegatorAllocationUpdated(_operatorDelegatorToRemove, 0);
174                     emit OperatorDelegatorRemoved(_operatorDelegatorToRemove);
253                     emit CollateralTokenRemoved(_collateralTokenToRemove);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/TimelockController.sol


273                 emit CallScheduled(id, i, targets[i], values[i], payloads[i], predecessor, delay);
360                 emit CallExecuted(id, i, target, value, payload);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


113                 emit WithdrawBufferTargetUpdated(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>

## G022 - Calling External Function With this Keyword:

Calling an external function internally, through the use of this wastes the gas overhead of calling an external function (100 gas). Instead, change the function from external to public, and remove the this.


```solidity
File: contracts/TimelockController.sol


417             return this.onERC721Received.selector;
430             return this.onERC1155Received.selector;
443             return this.onERC1155BatchReceived.selector;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

## G023 - `unchecked {}` can be used on the division of two `uints` in order to save gas:

The division cannot overflow, since both the numerator and the denominator are non-negative.


<details>
<summary>Click to show 6 findings</summary>

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


339                 (_price < lastPrice && (lastPrice - _price) > (lastPrice / 10))
280                 return (_amountIn * bridgeFeeShare) / FEE_BASIS;
386                 uint256 fee = (amountNextWETH * bridgeRouterFeeBps) / 10_000;
253             uint256 xezETHAmount = (1e18 * amountOut) / _lastPrice;
282                 return (sweepBatchSize * bridgeFeeShare) / FEE_BASIS;
338                 (_price > lastPrice && (_price - lastPrice) > (lastPrice / 10)) ||


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


241             bridges[_bridge].minterParams.ratePerSecond = _limit / _DURATION;
263             bridges[_bridge].burnerParams.ratePerSecond = _limit / _DURATION;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


167                 feeAmount = (msg.value * feeBasisPoints) / 10000;
261                     feeAmount = (balance * feeBasisPoints) / 10000;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


158             uint256 redeemAmount = (_currentValueInProtocol * _ezETHBeingBurned) / _existingEzETHSupply;
135             uint256 inflationPercentaage = (SCALE_FACTOR * _newValueAdded) /
136                 (_currentValueInProtocol + _newValueAdded);
139             uint256 newEzETHSupply = (_existingEzETHSupply * SCALE_FACTOR) /
140                 (SCALE_FACTOR - inflationPercentaage);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


40              return (10 ** 18 * totalTVL) / totalSupply;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


379                     (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
380                         BASIS_POINTS /
379                     (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
380                         BASIS_POINTS /
381                         BASIS_POINTS
421                     (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
422                         BASIS_POINTS /
423                         BASIS_POINTS &&
421                     (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
422                         BASIS_POINTS /


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

</details>

## G024 - Low level call can be optimized with assembly:

Low level call can be optimized with assembly. The returnData is copied to memory even if the variable is not utilized: the proper way to handle this is through a low level assembly call.


<details>
<summary>Click to show 6 findings</summary>

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


403             (bool success, ) = payable(msg.sender).call{ value: feeCollected }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


131                 (bool _success, ) = payable(_to).call{ value: _amount }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


520                 (bool success, ) = destination.call{ value: remainingAmount }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


168                 (bool success, ) = feeAddress.call{ value: feeAmount }("");
286             (bool success, ) = payable(msg.sender).call{ value: gasRefund }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


68              (bool success, ) = rewardDestination.call{ value: balance }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


369             (bool success, ) = target.call{ value: value }(data);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

</details>

## G025 - Use assembly to calculate hashes to save gas:

Using assembly to calculate hashes can save 80 gas per instance


<details>
<summary>Click to show 4 findings</summary>

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


147             bytes32 _salt = keccak256(abi.encodePacked(_name, _symbol, msg.sender));
190             bytes32 _salt = keccak256(abi.encodePacked(_xerc20, _baseToken, msg.sender));


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol


89              bytes32 _salt = keccak256(abi.encodePacked(_name, _symbol, msg.sender));


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L0:0

```solidity
File: contracts/Permissions/RoleManagerStorage.sol


11          bytes32 public constant RX_ETH_MINTER_BURNER = keccak256("RX_ETH_MINTER_BURNER");
14          bytes32 public constant OPERATOR_DELEGATOR_ADMIN = keccak256("OPERATOR_DELEGATOR_ADMIN");
17          bytes32 public constant ORACLE_ADMIN = keccak256("ORACLE_ADMIN");
20          bytes32 public constant RESTAKE_MANAGER_ADMIN = keccak256("RESTAKE_MANAGER_ADMIN");
23          bytes32 public constant TOKEN_ADMIN = keccak256("TOKEN_ADMIN");
26          bytes32 public constant NATIVE_ETH_RESTAKE_ADMIN = keccak256("NATIVE_ETH_RESTAKE_ADMIN");
29          bytes32 public constant ERC20_REWARD_ADMIN = keccak256("ERC20_REWARD_ADMIN");
32          bytes32 public constant DEPOSIT_WITHDRAW_PAUSER = keccak256("DEPOSIT_WITHDRAW_PAUSER");
39          bytes32 public constant BRIDGE_ADMIN = keccak256("BRIDGE_ADMIN");
42          bytes32 public constant PRICE_FEED_SENDER = keccak256("PRICE_FEED_SENDER");
47          bytes32 public constant WITHDRAW_QUEUE_ADMIN = keccak256("WITHDRAW_QUEUE_ADMIN");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManagerStorage.sol#L0:0

```solidity
File: contracts/TimelockController.sol


26          bytes32 public constant TIMELOCK_ADMIN_ROLE = keccak256("TIMELOCK_ADMIN_ROLE");
27          bytes32 public constant PROPOSER_ROLE = keccak256("PROPOSER_ROLE");
28          bytes32 public constant EXECUTOR_ROLE = keccak256("EXECUTOR_ROLE");
29          bytes32 public constant CANCELLER_ROLE = keccak256("CANCELLER_ROLE");
208             return keccak256(abi.encode(target, value, data, predecessor, salt));
222             return keccak256(abi.encode(targets, values, payloads, predecessor, salt));


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

</details>

## G026 - Unused named return variables without optimizer waste gas:

Consider changing the variable to be an unnamed one, since the variable is never assigned, nor is it returned by name. If the optimizer is not turned on, leaving the code as it is will also waste gas for the stack variable.


```solidity
File: contracts/Delegation/OperatorDelegator.sol


146         ) external nonReentrant onlyRestakeManager returns (uint256 shares) {
162         function _deposit(IERC20 _token, uint256 _tokenAmount) internal returns (uint256 shares) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


49              returns (
50                  uint80 roundId,
51                  int256 answer,
52                  uint256 startedAt,
53                  uint256 updatedAt,
54                  uint80 answeredInRound
55              )
72              returns (
73                  uint80 roundId,
74                  int256 answer,
75                  uint256 startedAt,
76                  uint256 updatedAt,
77                  uint80 answeredInRound
78              )


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


49              returns (
50                  uint80 roundId,
51                  int256 answer,
52                  uint256 startedAt,
53                  uint256 updatedAt,
54                  uint80 answeredInRound
55              )
72              returns (
73                  uint80 roundId,
74                  int256 answer,
75                  uint256 startedAt,
76                  uint256 updatedAt,
77                  uint80 answeredInRound
78              )


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

## G027 - Avoid updating storage when the value hasn't changed:

If the old value is equal to the new value, not re-storing the value will avoid a Gsreset (**2900 gas**), potentially at the expense of a Gcoldsload (**2100 gas**) or a Gwarmaccess (**100 gas**).


<details>
<summary>Click to show 7 findings</summary>

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


107         function updateCCIPEthChainSelector(uint64 _newChainSelector) external onlyOwner {
96          function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {
134         function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


103         function updateCCIPEthChainSelector(uint32 _newChainDomain) external onlyOwner {
92          function updateXRenzoBridgeL1(address _newXRenzoBridgeL1) external onlyOwner {
130         function setRenzoDeposit(address _newXRenzoDeposit) external onlyOwner {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


121         function setLockbox(address _lockbox) public {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


54          function initialize(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


44          function initialize(address _xerc20, address _erc20, bool _isNative) public initializer {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


35          function initialize(


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/TimelockController.sol


402         function updateDelay(uint256 newDelay) external virtual {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

</details>

## G028 - Do not calculate constants:

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas.


```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


13          uint256 public constant MAX_TIME_WINDOW = 86400 + 60; // 24 hours + 60 seconds


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


23          uint256 constant SCALE_FACTOR = 10 ** 18;
26          uint256 constant MAX_TIME_WINDOW = 86400 + 60; // 24 hours + 60 seconds


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

## G029 - Duplicated `require()`/`revert()` checks should be refactored to a modifier or function:

Saves deployment costs.


```solidity
File: contracts/TimelockController.sol


267             require(targets.length == values.length, "TimelockController: length mismatch");
268             require(targets.length == payloads.length, "TimelockController: length mismatch");
377             require(isOperationReady(id), "TimelockController: operation is not ready");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

## G030 - The use of a logical AND in place of double if is slightly less gas efficient in instances where there isn't a corresponding else statement for the given if statement:

Using a double if statement instead of logical AND (&&) can provide similar short-circuiting behavior whereas double if is slightly more efficient.


<details>
<summary>Click to show 5 findings</summary>

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


291             if (receiver == address(0) && address(oracle) == address(0)) revert PriceFeedNotAvailable();
337             if (
338                 (_price > lastPrice && (_price - lastPrice) > (lastPrice / 10)) ||


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


111             if ((_baseToken == address(0) && !_isNative) || (_isNative && _baseToken != address(0))) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


166             if (feeAddress != address(0x0) && feeBasisPoints > 0) {
260                 if (feeAddress != address(0x0) && feeBasisPoints > 0) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


419                 if (
420                     operatorDelegatorTVLs[i] >
421                     (operatorDelegatorAllocations[operatorDelegators[i]] * totalTVL) /
422                         BASIS_POINTS /
423                         BASIS_POINTS &&
424                     operatorDelegatorTokenTVLs[i][tokenIndex] >= ezETHValue
510             if (maxDepositTVL != 0 && totalTVL + collateralTokenValue > maxDepositTVL) {
597             if (maxDepositTVL != 0 && totalTVL + msg.value > maxDepositTVL) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


69              if (from != address(0) && to != address(0)) {


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## G031 - Use the inputs/results of assignments rather than re-reading state variables:

When a state variable is assigned, it saves gas to use the value being assigned, later in the function, rather than re-reading the state variable itself. If needed, it can also be stored to a local variable, and be used in that way. Both options avoid a Gwarmaccess (100 gas). Note that if the operation is, say +=, the assignment also results in a value which can be used. The instances below point to the first reference after the assignment, since later references are already covered by issues describing the caching of state variable values.


```solidity
File: contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol


69              address xerc20 = IXERC20Registry(registry).getXERC20(_erc20);
70              address lockbox = IXERC20Registry(registry).getLockbox(xerc20);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


72              if (_ccipSender != xRenzoBridgeL1) revert InvalidSender(xRenzoBridgeL1, _ccipSender);
75                  revert InvalidSourceChain(ccipEthChainSelector, _ccipSourceChainSelector);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

## G032 -  Initializers can be marked payable:

Payable functions cost less gas to execute, since the compiler does not have to add extra checks to ensure that a payment wasn't provided. An initializer can safely be marked as payable, since only the deployer would be able to pass funds, and the project itself would not pass any funds.


<details>
<summary>Click to show 18 findings</summary>

```solidity
File: contracts/Bridge/L1/xRenzoBridge.sol


70          function initialize(
71              IERC20 _ezETH,
72              IERC20 _xezETH,
73              IRestakeManager _restakeManager,
74              IERC20 _wETH,
75              IXERC20Lockbox _xezETHLockbox,
76              IConnext _connext,
77              IRouterClient _linkRouterClient,
78              IRateProvider _rateProvider,
79              LinkTokenInterface _linkToken,
80              IRoleManager _roleManager
81          ) public initializer {
82              // Verify non-zero addresses on inputs
83              if (
84                  address(_ezETH) == address(0) ||
85                  address(_xezETH) == address(0) ||
86                  address(_restakeManager) == address(0) ||
87                  address(_wETH) == address(0) ||
88                  address(_xezETHLockbox) == address(0) ||
89                  address(_connext) == address(0) ||
90                  address(_linkRouterClient) == address(0) ||
91                  address(_rateProvider) == address(0) ||
92                  address(_linkToken) == address(0) ||
93                  address(_roleManager) == address(0)
94              ) {
95                  revert InvalidZeroInput();
96              }
97      
98              // Verify all tokens have 18 decimals
99              uint8 decimals = IERC20MetadataUpgradeable(address(_ezETH)).decimals();
100             if (decimals != EXPECTED_DECIMALS) {
101                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
102             }
103             decimals = IERC20MetadataUpgradeable(address(_xezETH)).decimals();
104             if (decimals != EXPECTED_DECIMALS) {
105                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
106             }
107             decimals = IERC20MetadataUpgradeable(address(_wETH)).decimals();
108             if (decimals != EXPECTED_DECIMALS) {
109                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
110             }
111             decimals = IERC20MetadataUpgradeable(address(_linkToken)).decimals();
112             if (decimals != EXPECTED_DECIMALS) {
113                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
114             }
115     
116             // Save off inputs
117             ezETH = _ezETH;
118             xezETH = _xezETH;
119             restakeManager = _restakeManager;
120             wETH = _wETH;
121             xezETHLockbox = _xezETHLockbox;
122             connext = _connext;
123             linkRouterClient = _linkRouterClient;
124             rateProvider = _rateProvider;
125             linkToken = _linkToken;
126             roleManager = _roleManager;
127         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L1/xRenzoBridge.sol#L0:0

```solidity
File: contracts/Bridge/L2/Oracle/RenzoOracleL2.sol


23          function initialize(AggregatorV3Interface _oracle) public initializer {
24              // Initialize inherited classes
25              __Ownable_init();
26      
27              if (address(_oracle) == address(0)) revert InvalidZeroInput();
28      
29              // Verify that the pricing of the oracle less than or equal 18 decimals - pricing calculations will be off otherwise
30              if (_oracle.decimals() > 18) revert InvalidTokenDecimals(18, _oracle.decimals());
31      
32              oracle = _oracle;
33          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


75          function initialize(
76              uint256 _currentPrice,
77              IERC20 _xezETH,
78              IERC20 _depositToken,
79              IERC20 _collateralToken,
80              IConnext _connext,
81              bytes32 _swapKey,
82              address _receiver,
83              uint32 _bridgeDestinationDomain,
84              address _bridgeTargetAddress,
85              IRenzoOracleL2 _oracle
86          ) public initializer {
87              // Initialize inherited classes
88              __Ownable_init();
89      
90              // Verify valid non zero values
91              if (
92                  _currentPrice == 0 ||
93                  address(_xezETH) == address(0) ||
94                  address(_depositToken) == address(0) ||
95                  address(_collateralToken) == address(0) ||
96                  address(_connext) == address(0) ||
97                  _swapKey == 0 ||
98                  _bridgeDestinationDomain == 0 ||
99                  _bridgeTargetAddress == address(0)
100             ) {
101                 revert InvalidZeroInput();
102             }
103     
104             // Verify all tokens have 18 decimals
105             uint8 decimals = IERC20MetadataUpgradeable(address(_depositToken)).decimals();
106             if (decimals != EXPECTED_DECIMALS) {
107                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
108             }
109             decimals = IERC20MetadataUpgradeable(address(_collateralToken)).decimals();
110             if (decimals != EXPECTED_DECIMALS) {
111                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
112             }
113             decimals = IERC20MetadataUpgradeable(address(_xezETH)).decimals();
114             if (decimals != EXPECTED_DECIMALS) {
115                 revert InvalidTokenDecimals(EXPECTED_DECIMALS, decimals);
116             }
117     
118             // Initialize the price and timestamp
119             lastPrice = _currentPrice;
120             lastPriceTimestamp = block.timestamp;
121     
122             // Set xezETH address
123             xezETH = _xezETH;
124     
125             // Set the depoist token
126             depositToken = _depositToken;
127     
128             // Set the collateral token
129             collateralToken = _collateralToken;
130     
131             // Set the connext contract
132             connext = _connext;
133     
134             // Set the swap key
135             swapKey = _swapKey;
136     
137             // Set receiver contract address
138             receiver = _receiver;
139             // Connext router fee is 5 basis points
140             bridgeRouterFeeBps = 5;
141     
142             // Set the bridge destination domain
143             bridgeDestinationDomain = _bridgeDestinationDomain;
144     
145             // Set the bridge target address
146             bridgeTargetAddress = _bridgeTargetAddress;
147     
148             // set oracle Price Feed struct
149             oracle = _oracle;
150     
151             // set bridge Fee Share 0.05% where 100 basis point = 1%
152             bridgeFeeShare = 5;
153     
154             //set sweep batch size to 32 ETH
155             sweepBatchSize = 32 ether;
156         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20.sol


61          function initialize(
62              string memory _name,
63              string memory _symbol,
64              address _factory
65          ) public initializer {
66              __XERC20_init(_name, _symbol, _factory);
67          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Factory.sol


54          function initialize(
55              address _lockboxImplementation,
56              address _xerc20Implementation
57          ) public initializer {
58              lockboxImplementation = _lockboxImplementation;
59              xerc20Implementation = _xerc20Implementation;
60          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


44          function initialize(address _xerc20, address _erc20, bool _isNative) public initializer {
45              XERC20 = IXERC20(_xerc20);
46              ERC20 = IERC20(_erc20);
47              IS_NATIVE = _isNative;
48          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol


35          function initialize(
36              string memory _name,
37              string memory _symbol,
38              address _factory,
39              address _l1Token,
40              address _optimismBridge
41          ) public initializer {
42              __ERC165_init();
43              __XERC20_init(_name, _symbol, _factory);
44              l1Token = _l1Token;
45              optimismBridge = _optimismBridge;
46          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


74          function initialize(
75              IRoleManager _roleManager,
76              IStrategyManager _strategyManager,
77              IRestakeManager _restakeManager,
78              IDelegationManager _delegationManager,
79              IEigenPodManager _eigenPodManager
80          ) external initializer {
81              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
82              if (address(_strategyManager) == address(0x0)) revert InvalidZeroInput();
83              if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
84              if (address(_delegationManager) == address(0x0)) revert InvalidZeroInput();
85              if (address(_eigenPodManager) == address(0x0)) revert InvalidZeroInput();
86      
87              __ReentrancyGuard_init();
88      
89              roleManager = _roleManager;
90              strategyManager = _strategyManager;
91              restakeManager = _restakeManager;
92              delegationManager = _delegationManager;
93              eigenPodManager = _eigenPodManager;
94      
95              // Deploy new EigenPod
96              eigenPod = IEigenPod(eigenPodManager.createPod());
97          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


74          function initialize(IRoleManager _roleManager) public initializer {
75              __ReentrancyGuard_init();
76      
77              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
78      
79              roleManager = _roleManager;
80          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Oracle/Binance/WBETHShim.sol


23          function initialize(IStakedTokenV2 _wBETHToken) public initializer {
24              if (address(_wBETHToken) == address(0x0)) revert InvalidZeroInput();
25      
26              wBETHToken = _wBETHToken;
27          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Binance/WBETHShim.sol#L0:0

```solidity
File: contracts/Oracle/Mantle/METHShim.sol


23          function initialize(IMethStaking _methStaking) public initializer {
24              if (address(_methStaking) == address(0x0)) revert InvalidZeroInput();
25      
26              methStaking = _methStaking;
27          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/Mantle/METHShim.sol#L0:0

```solidity
File: contracts/Oracle/RenzoOracle.sol


44          function initialize(IRoleManager _roleManager) public initializer {
45              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
46      
47              __ReentrancyGuard_init();
48      
49              roleManager = _roleManager;
50          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Oracle/RenzoOracle.sol#L0:0

```solidity
File: contracts/Permissions/RoleManager.sol


22          function initialize(address roleManagerAdmin) public initializer {
23              if (address(roleManagerAdmin) == address(0x0)) revert InvalidZeroInput();
24      
25              __AccessControl_init();
26      
27              _grantRole(DEFAULT_ADMIN_ROLE, roleManagerAdmin);
28          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Permissions/RoleManager.sol#L0:0

```solidity
File: contracts/RateProvider/BalancerRateProvider.sol


17          function initialize(
18              IRestakeManager _restakeManager,
19              IERC20Upgradeable _ezETHToken
20          ) public initializer {
21              if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
22              if (address(_ezETHToken) == address(0x0)) revert InvalidZeroInput();
23      
24              restakeManager = _restakeManager;
25              ezETHToken = _ezETHToken;
26          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RateProvider/BalancerRateProvider.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


101         function initialize(
102             IRoleManager _roleManager,
103             IEzEthToken _ezETH,
104             IRenzoOracle _renzoOracle,
105             IStrategyManager _strategyManager,
106             IDelegationManager _delegationManager,
107             IDepositQueue _depositQueue
108         ) public initializer {
109             __ReentrancyGuard_init();
110     
111             roleManager = _roleManager;
112             ezETH = _ezETH;
113             renzoOracle = _renzoOracle;
114             strategyManager = _strategyManager;
115             delegationManager = _delegationManager;
116             depositQueue = _depositQueue;
117             paused = false;
118         }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


38          function initialize(IRoleManager _roleManager, address _rewardDestination) public initializer {
39              __ReentrancyGuard_init();
40      
41              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
42              if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();
43      
44              roleManager = _roleManager;
45              rewardDestination = _rewardDestination;
46      
47              emit RewardDestinationUpdated(_rewardDestination);
48          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


64          function initialize(
65              IRoleManager _roleManager,
66              IRestakeManager _restakeManager,
67              IEzEthToken _ezETH,
68              IRenzoOracle _renzoOracle,
69              uint256 _coolDownPeriod,
70              TokenWithdrawBuffer[] calldata _withdrawalBufferTarget
71          ) external initializer {
72              if (
73                  address(_roleManager) == address(0) ||
74                  address(_ezETH) == address(0) ||
75                  address(_renzoOracle) == address(0) ||
76                  address(_restakeManager) == address(0) ||
77                  _withdrawalBufferTarget.length == 0 ||
78                  _coolDownPeriod == 0
79              ) revert InvalidZeroInput();
80      
81              __Pausable_init();
82      
83              roleManager = _roleManager;
84              restakeManager = _restakeManager;
85              ezETH = _ezETH;
86              renzoOracle = _renzoOracle;
87              coolDownPeriod = _coolDownPeriod;
88              for (uint256 i = 0; i < _withdrawalBufferTarget.length; ) {
89                  if (
90                      _withdrawalBufferTarget[i].asset == address(0) ||
91                      _withdrawalBufferTarget[i].bufferAmount == 0
92                  ) revert InvalidZeroInput();
93                  withdrawalBufferTarget[_withdrawalBufferTarget[i].asset] = _withdrawalBufferTarget[i]
94                      .bufferAmount;
95                  unchecked {
96                      ++i;
97                  }
98              }
99          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

```solidity
File: contracts/token/EzEthToken.sol


33          function initialize(IRoleManager _roleManager) public initializer {
34              if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
35      
36              __ERC20_init("ezETH", "Renzo Restaked ETH");
37              roleManager = _roleManager;
38          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/token/EzEthToken.sol#L0:0

</details>

## G033 - Avoid fetching a low-level call's return data by using assembly:

Even if you don't assign the call's second return value, it still gets copied to memory. Use assembly instead to prevent this and save 159 gas.


<details>
<summary>Click to show 6 findings</summary>

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


403             (bool success, ) = payable(msg.sender).call{ value: feeCollected }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol


131                 (bool _success, ) = payable(_to).call{ value: _amount }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


520                 (bool success, ) = destination.call{ value: remainingAmount }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


168                 (bool success, ) = feeAddress.call{ value: feeAmount }("");
286             (bool success, ) = payable(msg.sender).call{ value: gasRefund }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/Rewards/RewardHandler.sol


68              (bool success, ) = rewardDestination.call{ value: balance }("");


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Rewards/RewardHandler.sol#L0:0

```solidity
File: contracts/TimelockController.sol


369             (bool success, ) = target.call{ value: value }(data);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

</details>

## G034 - Using msg globals directly, rather than caching the value, saves gas:

For example, use msg.sender directly rather than storing it to a local variable


```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


120             lastPriceTimestamp = block.timestamp;
120             lastPriceTimestamp = block.timestamp;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

## G035 - State variable read in a loop:

The state variable should be cached in and read from a local variable, or accumulated in a local variable then written to storage once outside of the loop, rather than reading/updating it on every iteration of the loop, which will replace each Gwarmaccess (100 gas) with a much cheaper stack read.


```solidity
File: contracts/Delegation/OperatorDelegator.sol


210                 if (address(tokens[i]) == IS_NATIVE) {
284                 if (address(tokens[i]) != IS_NATIVE) {
385                 stakedButNotVerifiedEth -= (validatorCurrentBalanceGwei * GWEI_TO_WEI);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/RestakeManager.sol


380                         BASIS_POINTS /
422                         BASIS_POINTS /


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManager.sol#L0:0

```solidity
File: contracts/TimelockController.sol


108                 _setupRole(PROPOSER_ROLE, proposers[i]);
114                 _setupRole(EXECUTOR_ROLE, executors[i]);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

## G036 - State variables only set in the constructor should be declared immutable:

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).  While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.


```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


12          address public connext;


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

## G039 - Structs can be packed into fewer storage slots:

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings


```solidity
File: contracts/Bridge/Connext/libraries/LibConnextStorage.sol


38      struct TransferInfo {
39          uint32 originDomain;
40          uint32 destinationDomain;
41          uint32 canonicalDomain;
42          address to;
43          address delegate;
44          bool receiveLocal;
45          bytes callData;
46          uint256 slippage;
47          address originSender;
48          uint256 bridgedAmt;
49          uint256 normalizedIn;
50          uint256 nonce;
51          bytes32 canonicalId;
52      }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/Connext/libraries/LibConnextStorage.sol#L0:0

```solidity
File: contracts/RestakeManagerStorage.sol


29          struct PendingWithdrawal {
30              uint256 ezETHToBurn;
31              address withdrawer;
32              IERC20 tokenToWithdraw;
33              uint256 tokenAmountToWithdraw;
34              IOperatorDelegator operatorDelegator;
35              bool completed;
36          }


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/RestakeManagerStorage.sol#L0:0

## G040 - Use local variables for emitting:

Use the function/modifier's local copy of the state variable, rather than incurring an extra Gwarmaccess (100 gas). In the unlikely event that the state variable hasn't already been used by the function/modifier, consider whether it is really necessary to include it in the event, given the fact that it incurs a Gcoldsload (2100 gas), or whether it can be passed in to or back out of the functions that do use it


<details>
<summary>Click to show 7 findings</summary>

```solidity
File: contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol


98              emit XRenzoBridgeL1Updated(_newXRenzoBridgeL1, xRenzoBridgeL1);
109             emit CCIPEthChainSelectorUpdated(_newChainSelector, ccipEthChainSelector);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/CCIPReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol


94              emit XRenzoBridgeL1Updated(_newXRenzoBridgeL1, xRenzoBridgeL1);
105             emit ConnextEthChainDomainUpdated(_newChainDomain, connextEthChainDomain);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L0:0

```solidity
File: contracts/Bridge/L2/xRenzoDeposit.sol


448             emit BridgeSwept(bridgeDestinationDomain, bridgeTargetAddress, msg.sender, balance);
448             emit BridgeSwept(bridgeDestinationDomain, bridgeTargetAddress, msg.sender, balance);
512             emit ReceiverPriceFeedUpdated(_receiver, receiver);
523             emit BridgeFeeShareUpdated(bridgeFeeShare, _newShare);
534             emit SweepBatchSizeUpdated(sweepBatchSize, _newBatchSize);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Bridge/L2/xRenzoDeposit.sol#L0:0

```solidity
File: contracts/Delegation/OperatorDelegator.sol


136             emit BaseGasAmountSpentUpdated(baseGasAmountSpent, _baseGasAmountSpent);
244                 delegateAddress,
318                 delegationManager.calculateWithdrawalRoot(withdrawal),


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Delegation/OperatorDelegator.sol#L0:0

```solidity
File: contracts/Deposits/DepositQueue.sol


171                 emit ProtocolFeesPaid(IERC20(address(0x0)), feeAmount, feeAddress);
264                     emit ProtocolFeesPaid(token, feeAmount, feeAddress);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Deposits/DepositQueue.sol#L0:0

```solidity
File: contracts/TimelockController.sol


404             emit MinDelayChange(_minDelay, newDelay);


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/TimelockController.sol#L0:0

```solidity
File: contracts/Withdraw/WithdrawQueue.sol


114                     withdrawalBufferTarget[_newBufferTarget[i].asset],
131             emit CoolDownPeriodUpdated(coolDownPeriod, _newCoolDownPeriod);
256                 withdrawRequestNonce,


```

https://github.com/code-423n4/2024-04-renzo/blob/main/contracts/Withdraw/WithdrawQueue.sol#L0:0

</details>


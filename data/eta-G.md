# Use already cached value instead of re-reading from calldata
Reading from calldata takes 3 Gas in other consecutive after first access. To avoid 1 calldata, we can use already cached value.

[xRenzoBridge.sol#L218-253](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L218C1-L253C53)

```solidity
218 for (uint256 i = 0; i < _destinationParam.length; ) {
219   Client.EVM2AnyMessage memory evm2AnyMessage = Client.EVM2AnyMessage({
     +  CCIPDestinationParam[] calldata _destinationParami = _destinationParam[i];
220  -  receiver: abi.encode(_destinationParam[i]._renzoReceiver), // ABI-encoded xRenzoDepsot contract address
     +  receiver: abi.encode(_destinationParami._renzoReceiver), // ABI-encoded xRenzoDepsot contract address
233  -  _destinationParam[i].destinationChainSelector,
     +  _destinationParami.destinationChainSelector,
245  -  _destinationParam[i].destinationChainSelector,
     +  _destinationParami.destinationChainSelector,
252  -  _destinationParam[i].destinationChainSelector,
+  _destinationParami.destinationChainSelector,
253  -  _destinationParam[i]._renzoReceiver,
     +  _destinationParami._renzoReceiver,
```
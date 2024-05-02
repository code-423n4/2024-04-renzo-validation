1) ### xRenzoBridgeStorageV1::IxRenzoBridge
IxRenzoBridge is an interface and hence does not have any storage, it is best to have the interface derive in the implementation contract instead of storage. The IxRenzoBridge should instead be derived in the implementation contract as below.

```
contract xRenzoBridge is
    IXReceiver,
    Initializable,
    ReentrancyGuardUpgradeable,
    xRenzoBridgeStorageV1,
==> IxRenzoBridge
{
```

2) ### EzEthToken::name() & symbol()
The values should ideally be returned from storage of ERC20Upgradeable, and not from the overrides.
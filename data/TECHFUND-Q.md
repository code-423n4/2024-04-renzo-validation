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


3) ### TimelockController::roles should be mutually exclusive
The proposer and executor are two mutually exclusive roles and hence a single account cannot have both the roles. The check is missing in the constructor. Similar check should be applied through self administration process.

4) ### XERC20::initialize() order of initialisation should be same as order of inheritance.

Below is the order of Inheritance

```
   contract XERC20 is
    Initializable,
    ERC20Upgradeable,
    OwnableUpgradeable,
    IXERC20,
    ERC20PermitUpgradeable
```

Below is the order of initialization
```
        __ERC20_init(_name, _symbol);
        __ERC20Permit_init(_name);
        __Ownable_init();
```

hence permit init should be called after Ownable init()

5) ### xRenzoBridge::sendPrice has hardcoded gas limit. 
sendPrice() function has hard coded limit as below

```
    Client.EVMExtraArgsV1({ gasLimit: 200_000 })
```
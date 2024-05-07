## L-01: Missing events arithmetic
Detect missing events for critical arithmetic parameters. Emit an event for critical parameter changes.

- Found in contracts/Delegation/OperatorDelegator.sol [Line: 358](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol#L358)
    ```
	stakedButNotVerifiedEth += msg.value;
    ```

## L-02: Missing checks for `address(0)` when assigning values to address state variables
Check for `address(0)` when assigning values to address state variables.

- Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 479](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol#L479)
    ```
	address(_to).transfer(_amount);
    ```
- Found in contracts/Bridge/xERC20/contracts/XERC20Factory.sol [Line: 58-59](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L58-L59)
    ```
	lockboxImplementation = _lockboxImplementation;
	xerc20Implementation = _xerc20Implementation;
    ```
- Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 138](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol#L138)
    ```
	receiver = _receiver;
    ```
- Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 149](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol#L149)
    ```
	oracle = _oracle;
    ```
- Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 513](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol#L513)
    ```
	receiver = _receiver;
    ```
- Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 86](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol#L86)
    ```
	FACTORY = _factory;
    ```
- Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 123](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol#L123)
    ```
	lockbox = _lockbox;
    ```
- Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 45](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L45)
    ```
	XERC20 = IXERC20(_xerc20);
    ```
- Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 46](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L46)
    ```
	ERC20 = IERC20(_erc20);
    ```
- Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 44](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L44)
    ```
        l1Token = _l1Token;
    ```
- Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 45](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L45)
    ```
	optimismBridge = _optimismBridge;
    ```
- Found in contracts/RestakeManager.sol [Line: 111](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L111)
    ```
	roleManager = _roleManager;
    ```
- Found in contracts/RestakeManager.sol [Line: 112](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L112)
    ```
	ezETH = _ezETH;
    ```
- Found in contracts/RestakeManager.sol [Line: 113](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L113)
    ```
	renzoOracle = _renzoOracle;
    ```
- Found in contracts/RestakeManager.sol [Line: 114](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L114)
    ```
	strategyManager = _strategyManager;
    ```
- Found in contracts/RestakeManager.sol [Line: 115](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol#L115)
    ```
	delegationManager = _delegationManager;
    ```

## NC-01: Missing inheritance
Detect missing inheritance. contract xRenzoDeposit should inherit from IRenzoOracleL2 (https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/Oracle/IRenzoOracleL2.sol#4-6).

- Found in contracts/Bridge/L2/xRenzoDeposit.sol [Line: 27-33](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol#L27-L33)
    ```
	contract xRenzoDeposit is
    ```

## NC-02: `public` functions not used internally could be marked `external`
Instead of marking a function as `public`, consider marking it as `external` if it is not used internally.

- Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 61](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol#L61)
    ```solidity
	function initialize(
    ```
- Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 96](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol#L96)
    ```solidity
	function mint(address _user, uint256 _amount) public virtual {
    ```
- Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 107](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol#L107)
    ```solidity
	function burn(address _user, uint256 _amount) public virtual {
    ```
- Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 121](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol#L121)
    ```solidity
	function setLockbox(address _lockbox) public {
    ```
- Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 152](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol#L152)
    ```solidity
	function mintingMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
    ```
- Found in contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 163](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol#L163)
    ```solidity
	function burningMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
    ```
- Found in contracts/Bridge/xERC20/contracts/XERC20Factory.sol [Line: 54](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L54)
    ```solidity
	function initialize(
    ```
- Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 44](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L44)
    ```solidity
	function initialize(address _xerc20, address _erc20, bool _isNative) public initializer {
    ```
- Found in contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 91](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L91)
    ```solidity
	function depositNativeTo(address _to) public payable {
    ```
- Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 35](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L35)
    ```solidity
	function initialize(
    ```
- Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 48](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L48)
    ```solidity
	function supportsInterface(
    ```
- Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 56](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L56)
    ```solidity
	function remoteToken() public view override returns (address) {
    ```
- Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 60](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L60)
    ```solidity
	function bridge() public view override returns (address) {
    ```
- Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 64](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L64)
    ```solidity
	function mint(address _to, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
    ```
- Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 68](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L68)
    ```solidity
	function burn(address _from, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
    ```
- Found in contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol [Line: 37](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L37)
    ```solidity
	function deployOptimismMintableXERC20(
    ```
- Found in contracts/TimelockController.sol [Line: 142](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol#L142)
    ```solidity
	function supportsInterface(
    ```
- Found in contracts/TimelockController.sol [Line: 234](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol#L234)
    ```solidity
	function schedule(
    ```
- Found in contracts/TimelockController.sol [Line: 259](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol#L259)
    ```solidity
	function scheduleBatch(
    ```
- Found in contracts/TimelockController.sol [Line: 296](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol#L296)
    ```solidity
	function cancel(bytes32 id) public virtual onlyRole(CANCELLER_ROLE) {
    ```
- Found in contracts/TimelockController.sol [Line: 315](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol#L315)
    ```solidity
	function execute(
    ```
- Found in contracts/TimelockController.sol [Line: 342](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol#L342)
    ```solidity
	function executeBatch(
    ```
- Found in contracts/TimelockController.sol [Line: 411](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol#L411)
    ```solidity
	function onERC721Received(
    ```
- Found in contracts/TimelockController.sol [Line: 423](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol#L423)
    ```solidity
	function onERC1155Received(
    ```
- Found in contracts/TimelockController.sol [Line: 436](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol#L436)
    ```solidity
	function onERC1155BatchReceived(
    ```
- Found in contracts/token/EzEthToken.sol [Line: 33](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/token/EzEthToken.sol#L33)
    ```solidity
	function initialize(IRoleManager _roleManager) public initializer {
    ```
- Found in contracts/token/EzEthToken.sol [Line: 77](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/token/EzEthToken.sol#L77)
    ```solidity
	function name() public view virtual override returns (string memory) {
    ```
- Found in contracts/token/EzEthToken.sol [Line: 85](https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/token/EzEthToken.sol#L85)
    ```solidity
	function symbol() public view virtual override returns (string memory) {
    ```

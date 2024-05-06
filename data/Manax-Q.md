## L-1: Unsafe ERC20 Operations should not be used

ERC20 functions may not behave as expected. For example: return values are not always meaningful. It is recommended to use OpenZeppelin's SafeERC20 library.

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol [Line: 241]
(contracts/Bridge/L1/xRenzoBridge.sol#L241)

	```solidity
	            linkToken.approve(address(linkRouterClient), fees);
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol [Line: 295]
(contracts/Bridge/L1/xRenzoBridge.sol#L295)

	```solidity
	        payable(_to).transfer(_amount);
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 479]
(contracts/Bridge/L2/xRenzoDeposit.sol#L479)

	```solidity
	        payable(_to).transfer(_amount);
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 268]
(contracts/Deposits/DepositQueue.sol#L268)

	```solidity
	            token.approve(address(restakeManager), balance - feeAmount);
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol [Line: 303]
(contracts/Withdraw/WithdrawQueue.sol#L303)

	```solidity
	            payable(msg.sender).transfer(_withdrawRequest.amountToRedeem);
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol [Line: 305]
(contracts/Withdraw/WithdrawQueue.sol#L305)

	```solidity
	            IERC20(_withdrawRequest.collateralToken).transfer(
	```

## L-2: `public` functions not used internally could be marked `external`

For functions not used internally but used externally instead of marking them as `public` consider marking it as `external`

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol#L70
(contracts/Bridge/L1/xRenzoBridge.sol#L70)

	```solidity
	    function initialize(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol [Line: 23]
(contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L23)

	```solidity
	    function initialize(AggregatorV3Interface _oracle) public initializer {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol [Line: 50]
(contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L50)

	```solidity
	    function getMintRate() public view returns (uint256, uint256) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 75]
(contracts/Bridge/L2/xRenzoDeposit.sol#L75)

	```solidity
	    function initialize(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 414]
(contracts/Bridge/L2/xRenzoDeposit.sol#L414)

	```solidity
	    function sweep() public payable nonReentrant {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 61]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L61)

	```solidity
	    function initialize(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 96](contracts/Bridge/xERC20/contracts/XERC20.sol#L96)

	```solidity
	    function mint(address _user, uint256 _amount) public virtual {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 107](contracts/Bridge/xERC20/contracts/XERC20.sol#L107)

	```solidity
	    function burn(address _user, uint256 _amount) public virtual {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 121]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L121)

	```solidity
	    function setLockbox(address _lockbox) public {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 152]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L152)

	```solidity
	    function mintingMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 163]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L163)

	```solidity
	    function burningMaxLimitOf(address _bridge) public view returns (uint256 _limit) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Factory.sol [Line: 54]
(contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L54)

	```solidity
	    function initialize(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 44]
(contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L44)

	```solidity
	    function initialize(address _xerc20, address _erc20, bool _isNative) public initializer {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 91]
(contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L91)

	```solidity
	    function depositNativeTo(address _to) public payable {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 35]
(contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L35)

	```solidity
	    function initialize(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 48]
(contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L48)

	```solidity
	    function supportsInterface(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 56]
(contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L56)

	```solidity
	    function remoteToken() public view override returns (address) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 60]
(contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L60)

	```solidity
	    function bridge() public view override returns (address) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 64]
(contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L64)

	```solidity
	    function mint(address _to, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 68]
(contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L68)

	```solidity
	    function burn(address _from, uint256 _amount) public override(XERC20, IOptimismMintableERC20) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol [Line: 37]
(contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L37)

	```solidity
	    function deployOptimismMintableXERC20(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol [Line: 172]
(contracts/Delegation/OperatorDelegator.sol#L172)

	```solidity
	    function getStrategyIndex(IStrategy _strategy) public view returns (uint256) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 74]
(contracts/Deposits/DepositQueue.sol#L74)

	```solidity
	    function initialize(IRoleManager _roleManager) public initializer {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/Binance/WBETHShim.sol [Line: 23]
(contracts/Oracle/Binance/WBETHShim.sol#L23)

	```solidity
	    function initialize(IStakedTokenV2 _wBETHToken) public initializer {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/Mantle/METHShim.sol [Line: 23]
(contracts/Oracle/Mantle/METHShim.sol#L23)

	```solidity
	    function initialize(IMethStaking _methStaking) public initializer {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol [Line: 44]
(contracts/Oracle/RenzoOracle.sol#L44)

	```solidity
	    function initialize(IRoleManager _roleManager) public initializer {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Permissions/RoleManager.sol [Line: 22]
(contracts/Permissions/RoleManager.sol#L22)

	```solidity
	    function initialize(address roleManagerAdmin) public initializer {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RateProvider/BalancerRateProvider.sol [Line: 17]
(contracts/RateProvider/BalancerRateProvider.sol#L17)

	```solidity
	    function initialize(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 101]
(contracts/RestakeManager.sol#L101)

	```solidity
	    function initialize(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 400]
(contracts/RestakeManager.sol#L400)

	```solidity
	    function chooseOperatorDelegatorForWithdraw(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Rewards/RewardHandler.sol [Line: 38]
(contracts/Rewards/RewardHandler.sol#L38)

	```solidity
	    function initialize(IRoleManager _roleManager, address _rewardDestination) public initializer {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol [Line: 142]
(contracts/TimelockController.sol#L142)

	```solidity
	    function supportsInterface(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol [Line: 234]
(contracts/TimelockController.sol#L234)

	```solidity
	    function schedule(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol [Line: 259]
(contracts/TimelockController.sol#L259)

	```solidity
	    function scheduleBatch(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol [Line: 296]
(contracts/TimelockController.sol#L296)

	```solidity
	    function cancel(bytes32 id) public virtual onlyRole(CANCELLER_ROLE) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol [Line: 315]
(contracts/TimelockController.sol#L315)

	```solidity
	    function execute(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol [Line: 342]
(contracts/TimelockController.sol#L342)

	```solidity
	    function executeBatch(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol [Line: 411]
(contracts/TimelockController.sol#L411)

	```solidity
	    function onERC721Received(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol [Line: 423]
(contracts/TimelockController.sol#L423)

	```solidity
	    function onERC1155Received(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol [Line: 436]
(contracts/TimelockController.sol#L436)

	```solidity
	    function onERC1155BatchReceived(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol [Line: 170]
(contracts/Withdraw/WithdrawQueue.sol#L170)

	```solidity
	    function getBufferDeficit(address _asset) public view returns (uint256) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Withdraw/WithdrawQueue.sol [Line: 270]
(contracts/Withdraw/WithdrawQueue.sol#L270)

	```solidity
	    function getOutstandingWithdrawRequests(address user) public view returns (uint256) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/token/EzEthToken.sol [Line: 33]
(contracts/token/EzEthToken.sol#L33)

	```solidity
	    function initialize(IRoleManager _roleManager) public initializer {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/token/EzEthToken.sol [Line: 77]
(contracts/token/EzEthToken.sol#L77)

	```solidity
	    function name() public view virtual override returns (string memory) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/token/EzEthToken.sol [Line: 85]
(contracts/token/EzEthToken.sol#L85)

	```solidity
	    function symbol() public view virtual override returns (string memory) {
	```

## L-3: Define and use `constant` variables instead of using MAGIC NUMBERS

If the same  value is used multiple times, create a constant state variable and reference it throughout the contract.

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol [Line: 30]
(contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L30)

	```solidity
	        if (_oracle.decimals() > 18) revert InvalidTokenDecimals(18, _oracle.decimals());
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol [Line: 39]
(contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L39)

	```solidity
	        if (_oracleAddress.decimals() > 18)
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol [Line: 40]
(contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L40)

	```solidity
	            revert InvalidTokenDecimals(18, _oracleAddress.decimals());
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/Oracle/RenzoOracleL2.sol [Line: 54]
(contracts/Bridge/L2/Oracle/RenzoOracleL2.sol#L54)

	```solidity
	        uint256 _scaledPrice = (uint256(price)) * 10 ** (18 - oracle.decimals());
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 140]
(contracts/Bridge/L2/xRenzoDeposit.sol#L140)

	```solidity
	        bridgeRouterFeeBps = 5;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 152]
(contracts/Bridge/L2/xRenzoDeposit.sol#L152)

	```solidity
	        bridgeFeeShare = 5;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 155]
(contracts/Bridge/L2/xRenzoDeposit.sol#L155)

	```solidity
	        sweepBatchSize = 32 ether;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 338]
(contracts/Bridge/L2/xRenzoDeposit.sol#L338)

	```solidity
	            (_price > lastPrice && (_price - lastPrice) > (lastPrice / 10)) ||
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 339]
(contracts/Bridge/L2/xRenzoDeposit.sol#L339)

	```solidity
	            (_price < lastPrice && (lastPrice - _price) > (lastPrice / 10))
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 533]
(contracts/Bridge/L2/xRenzoDeposit.sol#L533)

	```solidity
	        if (_newBatchSize < 32 ether) revert InvalidSweepBatchSize(_newBatchSize);
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol [Line: 81]
(contracts/Delegation/OperatorDelegator.sol#L81)

	```solidity
	        if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol [Line: 82]
(contracts/Delegation/OperatorDelegator.sol#L82)

	```solidity
	        if (address(_strategyManager) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol [Line: 83]
(contracts/Delegation/OperatorDelegator.sol#L83)

	```solidity
	        if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol [Line: 84]
(contracts/Delegation/OperatorDelegator.sol#L84)

	```solidity
	        if (address(_delegationManager) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol [Line: 85]
(contracts/Delegation/OperatorDelegator.sol#L85)

	```solidity
	        if (address(_eigenPodManager) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol [Line: 110]
(contracts/Delegation/OperatorDelegator.sol#L110)

	```solidity
	        if (address(_token) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol [Line: 122]
(contracts/Delegation/OperatorDelegator.sol#L122)

	```solidity
	        if (address(_delegateAddress) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol [Line: 123]
(contracts/Delegation/OperatorDelegator.sol#L123)

	```solidity
	        if (address(delegateAddress) != address(0x0)) revert DelegateAddressAlreadySet();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol [Line: 147]
(contracts/Delegation/OperatorDelegator.sol#L147)

	```solidity
	        if (address(tokenStrategyMapping[token]) == address(0x0) || tokenAmount == 0)
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 77]
(contracts/Deposits/DepositQueue.sol#L77)

	```solidity
	        if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 99]
(contracts/Deposits/DepositQueue.sol#L99)

	```solidity
	            if (_feeAddress == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 103]
(contracts/Deposits/DepositQueue.sol#L103)

	```solidity
	        if (_feeBasisPoints > 10000) revert OverMaxBasisPoints();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 113]
(contracts/Deposits/DepositQueue.sol#L113)

	```solidity
	        if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 166]
(contracts/Deposits/DepositQueue.sol#L166)

	```solidity
	        if (feeAddress != address(0x0) && feeBasisPoints > 0) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 167]
(contracts/Deposits/DepositQueue.sol#L167)

	```solidity
	            feeAmount = (msg.value * feeBasisPoints) / 10000;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 171]
(contracts/Deposits/DepositQueue.sol#L171)

	```solidity
	            emit ProtocolFeesPaid(IERC20(address(0x0)), feeAmount, feeAddress);
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 179]
(contracts/Deposits/DepositQueue.sol#L179)

	```solidity
	        totalEarned[address(0x0)] = totalEarned[address(0x0)] + remainingRewards;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 182]
(contracts/Deposits/DepositQueue.sol#L182)

	```solidity
	        emit RewardsDeposited(IERC20(address(0x0)), remainingRewards);
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 202]
(contracts/Deposits/DepositQueue.sol#L202)

	```solidity
	        emit ETHStakedFromQueue(operatorDelegator, pubkey, 32 ether, address(this).balance);
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 239]
(contracts/Deposits/DepositQueue.sol#L239)

	```solidity
	                32 ether,
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 260]
(contracts/Deposits/DepositQueue.sol#L260)

	```solidity
	            if (feeAddress != address(0x0) && feeBasisPoints > 0) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 261]
(contracts/Deposits/DepositQueue.sol#L261)

	```solidity
	                feeAmount = (balance * feeBasisPoints) / 10000;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 129]
(contracts/EigenLayer/libraries/BeaconChainProofs.sol#L129)

	```solidity
	                32 * ((VALIDATOR_TREE_HEIGHT + 1) + BEACON_STATE_FIELD_TREE_HEIGHT),
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 162]
(contracts/EigenLayer/libraries/BeaconChainProofs.sol#L162)

	```solidity
	            stateRootProof.length == 32 * (BEACON_BLOCK_HEADER_FIELD_TREE_HEIGHT),
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 214]
(contracts/EigenLayer/libraries/BeaconChainProofs.sol#L214)

	```solidity
	                32 * (executionPayloadHeaderFieldTreeHeight + WITHDRAWALS_TREE_HEIGHT + 1),
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 219]
(contracts/EigenLayer/libraries/BeaconChainProofs.sol#L219)

	```solidity
	                32 * (BEACON_BLOCK_HEADER_FIELD_TREE_HEIGHT + BEACON_BLOCK_BODY_FIELD_TREE_HEIGHT),
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 223]
(contracts/EigenLayer/libraries/BeaconChainProofs.sol#L223)

	```solidity
	            withdrawalProof.slotProof.length == 32 * (BEACON_BLOCK_HEADER_FIELD_TREE_HEIGHT),
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 227]
(contracts/EigenLayer/libraries/BeaconChainProofs.sol#L227)

	```solidity
	            withdrawalProof.timestampProof.length == 32 * (executionPayloadHeaderFieldTreeHeight),
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 233]
(contracts/EigenLayer/libraries/BeaconChainProofs.sol#L233)

	```solidity
	                32 *
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/BytesLib.sol [Line: 370]
(contracts/EigenLayer/libraries/BytesLib.sol#L370)

	```solidity
	        require(_bytes.length >= _start + 32, "toUint256_outOfBounds");
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/BytesLib.sol [Line: 381]
(contracts/EigenLayer/libraries/BytesLib.sol#L381)

	```solidity
	        require(_bytes.length >= _start + 32, "toBytes32_outOfBounds");
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Endian.sol [Line: 16]
(contracts/EigenLayer/libraries/Endian.sol#L16)

	```solidity
	            (n >> 56) |
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Endian.sol [Line: 17]
(contracts/EigenLayer/libraries/Endian.sol#L17)

	```solidity
	            ((0x00FF000000000000 & n) >> 40) |
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Endian.sol [Line: 18]
(contracts/EigenLayer/libraries/Endian.sol#L18)

	```solidity
	            ((0x0000FF0000000000 & n) >> 24) |
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Endian.sol [Line: 19]
(contracts/EigenLayer/libraries/Endian.sol#L19)

	```solidity
	            ((0x000000FF00000000 & n) >> 8) |
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Endian.sol [Line: 20]
(contracts/EigenLayer/libraries/Endian.sol#L20)

	```solidity
	            ((0x00000000FF000000 & n) << 8) |
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Endian.sol [Line: 21]
(contracts/EigenLayer/libraries/Endian.sol#L21)

	```solidity
	            ((0x0000000000FF0000 & n) << 24) |
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Endian.sol [Line: 22]
(contracts/EigenLayer/libraries/Endian.sol#L22)

	```solidity
	            ((0x000000000000FF00 & n) << 40) |
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Endian.sol [Line: 23]
(contracts/EigenLayer/libraries/Endian.sol#L23)

	```solidity
	            ((0x00000000000000FF & n) << 56);
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Merkle.sol [Line: 54]
(contracts/EigenLayer/libraries/Merkle.sol#L54)

	```solidity
	            proof.length != 0 && proof.length % 32 == 0,
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Merkle.sol [Line: 58]
(contracts/EigenLayer/libraries/Merkle.sol#L58)

	```solidity
	        for (uint256 i = 32; i <= proof.length; i += 32) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Merkle.sol [Line: 113]
(contracts/EigenLayer/libraries/Merkle.sol#L113)

	```solidity
	            proof.length != 0 && proof.length % 32 == 0,
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Merkle.sol [Line: 117]
(contracts/EigenLayer/libraries/Merkle.sol#L117)

	```solidity
	        for (uint256 i = 32; i <= proof.length; i += 32) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol [Line: 45]
(contracts/Oracle/RenzoOracle.sol#L45)

	```solidity
	        if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol [Line: 58]
(contracts/Oracle/RenzoOracle.sol#L58)

	```solidity
	        if (address(_token) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol [Line: 61]
(contracts/Oracle/RenzoOracle.sol#L61)

	```solidity
	        if (_oracleAddress.decimals() != 18)
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol [Line: 62]
(contracts/Oracle/RenzoOracle.sol#L62)

	```solidity
	            revert InvalidTokenDecimals(18, _oracleAddress.decimals());
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol [Line: 73]
(contracts/Oracle/RenzoOracle.sol#L73)

	```solidity
	        if (address(oracle) == address(0x0)) revert OracleNotFound();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol [Line: 90]
(contracts/Oracle/RenzoOracle.sol#L90)

	```solidity
	        if (address(oracle) == address(0x0)) revert OracleNotFound();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RateProvider/BalancerRateProvider.sol [Line: 21]
(contracts/RateProvider/BalancerRateProvider.sol#L21)

	```solidity
	        if (address(_restakeManager) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RateProvider/BalancerRateProvider.sol [Line: 22]
(contracts/RateProvider/BalancerRateProvider.sol#L22)

	```solidity
	        if (address(_ezETHToken) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 146]
(contracts/RestakeManager.sol#L146)

	```solidity
	        if (_allocationBasisPoints > (100 * BASIS_POINTS)) revert OverMaxBasisPoints();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 191]
(contracts/RestakeManager.sol#L191)

	```solidity
	        if (address(_operatorDelegator) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 192]
(contracts/RestakeManager.sol#L192)

	```solidity
	        if (_allocationBasisPoints > (100 * BASIS_POINTS)) revert OverMaxBasisPoints();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 231]
(contracts/RestakeManager.sol#L231)

	```solidity
	        if (IERC20Metadata(address(_newCollateralToken)).decimals() != 18)
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 233]
(contracts/RestakeManager.sol#L233)

	```solidity
	                18,
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 615]
(contracts/RestakeManager.sol#L615)

	```solidity
	        emit Deposit(msg.sender, IERC20(address(0x0)), msg.value, ezETHToMint, _referralId);
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 679]
(contracts/RestakeManager.sol#L679)

	```solidity
	        totalRewards += depositQueue.totalEarned(address(0x0));
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Rewards/RewardHandler.sol [Line: 41]
(contracts/Rewards/RewardHandler.sol#L41)

	```solidity
	        if (address(_roleManager) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Rewards/RewardHandler.sol [Line: 42]
(contracts/Rewards/RewardHandler.sol#L42)

	```solidity
	        if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Rewards/RewardHandler.sol [Line: 75]
(contracts/Rewards/RewardHandler.sol#L75)

	```solidity
	        if (address(_rewardDestination) == address(0x0)) revert InvalidZeroInput();
	```

## L-4: The `nonReentrant` `modifier` should occur before all other modifiers

Put the nonReentrant modifier before any modifier as it is best-practice to protect against reentrancy in other modifiers.

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol [Line: 213](contracts/Bridge/L1/xRenzoBridge.sol#L213)

	```solidity
	    ) external payable onlyPriceFeedSender nonReentrant {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 216](contracts/Deposits/DepositQueue.sol#L216)

	```solidity
	    ) external onlyNativeEthRestakeAdmin nonReentrant {
	```
## L-5: Modifiers invoked only once can be shoe-horned into the function


- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L1/xRenzoBridge.sol [Line: 55]
(contracts/Bridge/L1/xRenzoBridge.sol#L55)

	```solidity
	    modifier onlyPriceFeedSender() {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol [Line: 43]
(contracts/Bridge/L2/PriceFeed/ConnextReceiver.sol#L43)

	```solidity
	    modifier onlySource(address _originSender, uint32 _origin) {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 62]
(contracts/Deposits/DepositQueue.sol#L62)

	```solidity
	    modifier onlyERC20RewardsAdmin() {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Oracle/RenzoOracle.sol [Line: 29]
(contracts/Oracle/RenzoOracle.sol#L29)

	```solidity
	    modifier onlyOracleAdmin() {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 77]
(contracts/RestakeManager.sol#L77)

	```solidity
	    modifier onlyDepositWithdrawPauserAdmin() {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Rewards/RewardHandler.sol [Line: 18]
(contracts/Rewards/RewardHandler.sol#L18)

	```solidity
	    modifier onlyNativeEthRestakeAdmin() {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Rewards/RewardHandler.sol [Line: 24]
(contracts/Rewards/RewardHandler.sol#L24)

	```solidity
	    modifier onlyRestakeManagerAdmin() {
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/token/EzEthToken.sol [Line: 21]
(contracts/token/EzEthToken.sol#L21)

	```solidity
	    modifier onlyTokenAdmin() {
	```

## L-6: Large values multiples of 10000 can be replaced with scientific notation

Use `e` notation, for example: `1e18`, instead of its full numeric value.

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 40]
(contracts/Bridge/L2/xRenzoDeposit.sol#L40)

	```solidity
	    uint32 public constant FEE_BASIS = 10000;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 103]
(contracts/Deposits/DepositQueue.sol#L103)

	```solidity
	        if (_feeBasisPoints > 10000) revert OverMaxBasisPoints();
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 167]
(contracts/Deposits/DepositQueue.sol#L167)

	```solidity
	            feeAmount = (msg.value * feeBasisPoints) / 10000;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 261]
(contracts/Deposits/DepositQueue.sol#L261)

	```solidity
	                feeAmount = (balance * feeBasisPoints) / 10000;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Endian.sol [Line: 17]
(contracts/EigenLayer/libraries/Endian.sol#L17)

	```solidity
	            ((0x00FF000000000000 & n) >> 40) |
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Endian.sol [Line: 18]
(contracts/EigenLayer/libraries/Endian.sol#L18)

	```solidity
	            ((0x0000FF0000000000 & n) >> 24) |
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Endian.sol [Line: 19]
(contracts/EigenLayer/libraries/Endian.sol#L19)

	```solidity
	            ((0x000000FF00000000 & n) >> 8) |
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Endian.sol [Line: 20]
(contracts/EigenLayer/libraries/Endian.sol#L20)

	```solidity
	            ((0x00000000FF000000 & n) << 8) |
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Endian.sol [Line: 21]
(contracts/EigenLayer/libraries/Endian.sol#L21)

	```solidity
	            ((0x0000000000FF0000 & n) << 24) |
	```

## L-7: Internal functions called only once can be inlined

Instead of separating the logic into a separate function, consider inlining the logic into the calling function and explain what the logic does in comment. This can reduce the number of function calls and improve readability.

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/BeaconChainProofs.sol [Line: 339]
(contracts/EigenLayer/libraries/BeaconChainProofs.sol#L339)

	```solidity
	    function getWithdrawalTimestamp(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Merkle.sol [Line: 48]
(contracts/EigenLayer/libraries/Merkle.sol#L48)

	```solidity
	    function processInclusionProofKeccak(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Merkle.sol [Line: 107]
(contracts/EigenLayer/libraries/Merkle.sol#L107)

	```solidity
	    function processInclusionProofSha256(
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/StructuredLinkedList.sol [Line: 161]
(contracts/EigenLayer/libraries/StructuredLinkedList.sol#L161)

	```solidity
	    function remove(List storage self, uint256 _node) internal returns (uint256) {
	```

## L-8: Inconsistency in declaring uint256/uint variables within a contract

Consider keeping the naming convention consistent in a given contract and mention number of bit used for it

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol [Line: 17]
(contracts/Delegation/OperatorDelegator.sol#L37-38)

	```solidity
        event WithdrawStarted(
            bytes32 withdrawRoot,
            address staker,
            address delegatedTo,
            address withdrawer,
@>          uint nonce,
@>          uint startBlock,
            IStrategy[] strategies,
            uint256[] shares
        );
       ```

## L-9: Missing checks for `address(0)` when assigning values to address state variables

Check for `address(0)` when assigning values to address state variables.

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 138]
(contracts/Bridge/L2/xRenzoDeposit.sol#L138)

	```solidity
	        receiver = _receiver;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 149]
(contracts/Bridge/L2/xRenzoDeposit.sol#L149)

	```solidity
	        oracle = _oracle;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 467]
(contracts/Bridge/L2/xRenzoDeposit.sol#L467)

	```solidity
	        allowedBridgeSweepers[_sweeper] = _allowed;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 503]
(contracts/Bridge/L2/xRenzoDeposit.sol#L503)

	```solidity
	        oracle = _oracle;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/L2/xRenzoDeposit.sol [Line: 513]
(contracts/Bridge/L2/xRenzoDeposit.sol#L513)

	```solidity
	        receiver = _receiver;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 86]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L86)

	```solidity
	        FACTORY = _factory;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 123]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L123)

	```solidity
	        lockbox = _lockbox;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 208]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L208)

	```solidity
	        bridges[_bridge].minterParams.currentLimit = _currentLimit - _change;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 220]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L220)

	```solidity
	        bridges[_bridge].burnerParams.currentLimit = _currentLimit - _change;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 233]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L233)

	```solidity
	        bridges[_bridge].minterParams.maxLimit = _limit;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 235]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L235)

	```solidity
	        bridges[_bridge].minterParams.currentLimit = _calculateNewCurrentLimit(
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 241]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L241)

	```solidity
	        bridges[_bridge].minterParams.ratePerSecond = _limit / _DURATION;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 255]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L255)

	```solidity
	        bridges[_bridge].burnerParams.maxLimit = _limit;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 257]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L257)

	```solidity
	        bridges[_bridge].burnerParams.currentLimit = _calculateNewCurrentLimit(
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 263]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L263)

	```solidity
	        bridges[_bridge].burnerParams.ratePerSecond = _limit / _DURATION;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Factory.sol [Line: 58]
(contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L58)

	```solidity
	        lockboxImplementation = _lockboxImplementation;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Factory.sol [Line: 59]
(contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L59)

	```solidity
	        xerc20Implementation = _xerc20Implementation;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 45]
(contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L45)

	```solidity
	        XERC20 = IXERC20(_xerc20);
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 46]
(contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L46)

	```solidity
	        ERC20 = IERC20(_erc20);
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 44]
(contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L44)

	```solidity
	        l1Token = _l1Token;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 45]
(contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L45)

	```solidity
	        optimismBridge = _optimismBridge;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Delegation/OperatorDelegator.sol [Line: 112]
(contracts/Delegation/OperatorDelegator.sol#L112)

	```solidity
	        tokenStrategyMapping[_token] = _strategy;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Deposits/DepositQueue.sol [Line: 272]
(contracts/Deposits/DepositQueue.sol#L272)

	```solidity
	            totalEarned[address(token)] = totalEarned[address(token)] + balance - feeAmount;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 111]
(contracts/RestakeManager.sol#L111)

	```solidity
	        roleManager = _roleManager;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 112]
(contracts/RestakeManager.sol#L112)

	```solidity
	        ezETH = _ezETH;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 113]
(contracts/RestakeManager.sol#L113)

	```solidity
	        renzoOracle = _renzoOracle;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 114]
(contracts/RestakeManager.sol#L114)

	```solidity
	        strategyManager = _strategyManager;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 115]
(contracts/RestakeManager.sol#L115)

	```solidity
	        delegationManager = _delegationManager;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 116]
(contracts/RestakeManager.sol#L116)

	```solidity
	        depositQueue = _depositQueue;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 154]
(contracts/RestakeManager.sol#L154)

	```solidity
	        operatorDelegatorAllocations[_newOperatorDelegator] = _allocationBasisPoints;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 209]
(contracts/RestakeManager.sol#L209)

	```solidity
	        operatorDelegatorAllocations[_operatorDelegator] = _allocationBasisPoints;
	```

- Found in  https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/RestakeManager.sol [Line: 714]
(contracts/RestakeManager.sol#L714)

	```solidity
	        collateralTokenTvlLimits[_token] = _limit;
	```

## L-10: Solidity pragma should be specific, not wide

Consider using a specific version of Solidity in your contracts instead of a wide version. For example, instead of `pragma solidity ^0.8.0;`, use `pragma solidity 0.8.0;`

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IConnext.sol [Line: 2]
(contracts/Bridge/Connext/core/IConnext.sol#L2)

	```solidity
	pragma solidity ^0.8.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IWeth.sol [Line: 2]
(contracts/Bridge/Connext/core/IWeth.sol#L2)

	```solidity
	pragma solidity ^0.8.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/core/IXReceiver.sol [Line: 2]
(contracts/Bridge/Connext/core/IXReceiver.sol#L2)

	```solidity
	pragma solidity ^0.8.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol [Line: 2]
(contracts/Bridge/Connext/integration/LockboxAdapterBlast.sol#L2)

	```solidity
	pragma solidity ^0.8.13;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/libraries/LibConnextStorage.sol [Line: 2]
(contracts/Bridge/Connext/libraries/LibConnextStorage.sol#L2)

	```solidity
	pragma solidity ^0.8.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/Connext/libraries/TokenId.sol [Line: 2]
(contracts/Bridge/Connext/libraries/TokenId.sol#L2)

	```solidity
	pragma solidity ^0.8.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20.sol [Line: 2]
(contracts/Bridge/xERC20/contracts/XERC20.sol#L2)

	```solidity
	pragma solidity >=0.8.4 <0.9.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Factory.sol [Line: 2]
(contracts/Bridge/xERC20/contracts/XERC20Factory.sol#L2)

	```solidity
	pragma solidity >=0.8.4 <0.9.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol [Line: 2]
(contracts/Bridge/xERC20/contracts/XERC20Lockbox.sol#L2)

	```solidity
	pragma solidity >=0.8.4 <0.9.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol [Line: 2]
(contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20.sol#L2)

	```solidity
	pragma solidity >=0.8.4 <0.9.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol [Line: 2]
(contracts/Bridge/xERC20/contracts/optimism/OptimismMintableXERC20Factory.sol#L2)

	```solidity
	pragma solidity >=0.8.4 <0.9.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IOptimismMintableERC20.sol [Line: 2]
(contracts/Bridge/xERC20/interfaces/IOptimismMintableERC20.sol#L2)

	```solidity
	pragma solidity >=0.8.4 <0.9.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IXERC20.sol [Line: 2]
(contracts/Bridge/xERC20/interfaces/IXERC20.sol#L2)

	```solidity
	pragma solidity >=0.8.4 <0.9.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IXERC20Factory.sol [Line: 2]
(contracts/Bridge/xERC20/interfaces/IXERC20Factory.sol#L2)

	```solidity
	pragma solidity >=0.8.4 <0.9.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/Bridge/xERC20/interfaces/IXERC20Lockbox.sol [Line: 2]
(contracts/Bridge/xERC20/interfaces/IXERC20Lockbox.sol#L2)

	```solidity
	pragma solidity >=0.8.4 <0.9.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/IAVSDirectory.sol [Line: 2]
(contracts/EigenLayer/interfaces/IAVSDirectory.sol#L2)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/IBeaconChainOracle.sol [Line: 2]
(contracts/EigenLayer/interfaces/IBeaconChainOracle.sol#L2)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/IDelayedWithdrawalRouter.sol [Line: 2]
(contracts/EigenLayer/interfaces/IDelayedWithdrawalRouter.sol#L2)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/IDelegationFaucet.sol [Line: 2]
(contracts/EigenLayer/interfaces/IDelegationFaucet.sol#L2)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/IDelegationManager.sol [Line: 2]
(contracts/EigenLayer/interfaces/IDelegationManager.sol#L2)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/IETHPOSDeposit.sol [Line: 12]
(contracts/EigenLayer/interfaces/IETHPOSDeposit.sol#L12)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/IEigenPod.sol [Line: 2]
(contracts/EigenLayer/interfaces/IEigenPod.sol#L2)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/IEigenPodManager.sol [Line: 2]
(contracts/EigenLayer/interfaces/IEigenPodManager.sol#L2)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/IPausable.sol [Line: 2]
(contracts/EigenLayer/interfaces/IPausable.sol#L2)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/IPauserRegistry.sol [Line: 2]
(contracts/EigenLayer/interfaces/IPauserRegistry.sol#L2)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/ISignatureUtils.sol [Line: 2]
(contracts/EigenLayer/interfaces/ISignatureUtils.sol#L2)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/ISlasher.sol [Line: 2]
(contracts/EigenLayer/interfaces/ISlasher.sol#L2)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/IStrategy.sol [Line: 2]
(contracts/EigenLayer/interfaces/IStrategy.sol#L2)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/IStrategyManager.sol [Line: 2]
(contracts/EigenLayer/interfaces/IStrategyManager.sol#L2)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/interfaces/IWhitelister.sol [Line: 2]
(contracts/EigenLayer/interfaces/IWhitelister.sol#L2)

	```solidity
	pragma solidity >=0.5.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/BytesLib.sol [Line: 9]
(contracts/EigenLayer/libraries/BytesLib.sol#L9)

	```solidity
	pragma solidity >=0.8.0 <0.9.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Endian.sol [Line: 2]
(contracts/EigenLayer/libraries/Endian.sol#L2)

	```solidity
	pragma solidity ^0.8.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/EigenLayer/libraries/Merkle.sol [Line: 4]
(contracts/EigenLayer/libraries/Merkle.sol#L4)

	```solidity
	pragma solidity ^0.8.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/TimelockController.sol [Line: 4]
(contracts/TimelockController.sol#L4)

	```solidity
	pragma solidity ^0.8.0;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/token/EzEthToken.sol [Line: 2]
(contracts/token/EzEthToken.sol#L2)

	```solidity
	pragma solidity ^0.8.9;
	```

- Found in https://github.com/code-423n4/2024-04-renzo/blob/519e518f2d8dec9acf6482b84a181e403070d22d/contracts/token/IEzEthToken.sol [Line: 2]
(contracts/token/IEzEthToken.sol#L2)

	```solidity
	pragma solidity ^0.8.9;
	```
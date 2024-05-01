The protocol should include the calling of external function renzoOracle.calculateRedeemAmount(), in the else statement to save gas. This helps in not calling redundant function.
```solidity
    /**
     * @notice  Creates a withdraw request for user
     * @param   _amount  amount of ezETH to withdraw
     * @param   _assetOut  output token to receive on claim
     */
    //@audit Seems that _assetOut can use a different asset address of any tokens
    function withdraw(uint256 _amount, address _assetOut) external nonReentrant {
        // check for 0 values
        if (_amount == 0 || _assetOut == address(0)) revert InvalidZeroInput();

        // check if provided assetOut is supported
        if (withdrawalBufferTarget[_assetOut] == 0) revert UnsupportedWithdrawAsset();

        // transfer ezETH tokens to this address
        IERC20(address(ezETH)).safeTransferFrom(msg.sender, address(this), _amount);

        // calculate totalTVL
        (, , uint256 totalTVL) = restakeManager.calculateTVLs();
        // @audit this should be included in the below else statement
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
```

Mitigation
```diff
    /**
     * @notice  Creates a withdraw request for user
     * @param   _amount  amount of ezETH to withdraw
     * @param   _assetOut  output token to receive on claim
     */
    //@audit Seems that _assetOut can use a different asset address of any tokens
    function withdraw(uint256 _amount, address _assetOut) external nonReentrant {
        // check for 0 values
        if (_amount == 0 || _assetOut == address(0)) revert InvalidZeroInput();

        // check if provided assetOut is supported
        if (withdrawalBufferTarget[_assetOut] == 0) revert UnsupportedWithdrawAsset();

        // transfer ezETH tokens to this address
        IERC20(address(ezETH)).safeTransferFrom(msg.sender, address(this), _amount);

        // calculate totalTVL
        (, , uint256 totalTVL) = restakeManager.calculateTVLs();
        // @audit this should be included in the below else statement
        // Calculate amount to Redeem in ETH
--        uint256 amountToRedeem = renzoOracle.calculateRedeemAmount(
--            _amount,
--            ezETH.totalSupply(),
--            totalTVL
        );

        // update amount in claim asset, if claim asset is not ETH
        if (_assetOut != IS_NATIVE) {
            // Get ERC20 asset equivalent amount
            amountToRedeem = renzoOracle.lookupTokenAmountFromValue(
                IERC20(_assetOut),
                amountToRedeem
            );
++        } else {
++        uint256 amountToRedeem = renzoOracle.calculateRedeemAmount(
++            _amount,
++            ezETH.totalSupply(),
++            totalTVL
++        );
}
```
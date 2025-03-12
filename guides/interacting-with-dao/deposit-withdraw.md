# Deposit/Withdraw

The initial step to interact with the DAO involves depositing your assets. Initially, you must grant approval for the transfer of your funds by the `GovUserKeeper` contract. For simplicity, all funds are approved for transfer. Subsequently, the deposit method on `GovPool` is invoked, crediting your funds to the internal personal balance, thereby enabling you to utilize them within the DAO.

:warning: _All token amounts related inputs and outputs are expected to be in 18 decimals, regardless of the actual token's decimals._

```solidity
function deposit(IGovPool govPool) external {
    (, address govUserKeeperAddress, , , ) = govPool.getHelperContracts();
    IGovUserKeeper govUserKeeper = IGovUserKeeper(govUserKeeperAddress);

    IERC20(govUserKeeper.tokenAddress()).approve(govUserKeeperAddress, type(uint256).max);
    IERC721(govUserKeeper.nftAddress()).setApprovalForAll(govUserKeeperAddress, true);

    uint256 amount = 10 ether;
    uint256[] memory nftIds = new uint256[](3);
    nftIds[0] = 1;
    nftIds[1] = 2;
    nftIds[3] = 3;

    govPool.deposit(amount, nftIds);
}
```

:warning: _Both `tokenAddress` and `nftAddress` can possibly be zero._

To refund your assets, you can simply call the reverse `withdraw` method, passing the receiver as the first parameter. You can only withdraw assets that aren't locked in proposals or delegated to anyone. To withdraw such funds, you must first cancel your vote or undelegate, respectively

```solidity
    function withdraw(IGovPool govPool) external {
        (uint256 amount, uint256[] memory nftIds) = govPool.getWithdrawableAssets(address(this));

        govPool.withdraw(address(this), amount, nftIds);
    }
```

:warning: Deposits and withdrawals cannot be made within the same block (via multicall). This is done to prevent possible flashloan attacks.

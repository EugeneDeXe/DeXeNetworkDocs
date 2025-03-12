# Token sale proposal

Another interesting proposal that can be created within a DAO is the `TokenSaleProposal`. This proposal enables the initiation of token sales (a.k.a. tiers) according to certain rules. The main executor should be the `TokenSaleProposal` contract, which retains all the created tiers within a DAO it deployed alongside with. Both custom tokens and `ERC20Gov` tokens can be sold in tiers. Upon the creation of tiers, sale token amounts are transferred from the treasury to the `TokenSaleProposal` contract, so it's necessary to approve this transfer beforehand. Let's create a tier that will sell `ERC20Gov` tokens in exchange for the native currency.

```solidity
function createTier(
    IGovPool govPool,
    ITokenSaleProposal tokenSaleProposal,
    IERC20Gov govToken
) external {
    address[] memory purchaseTokenAddresses = new address[](1);
    uint256[] memory exchangeRates = new uint256[](1);

    /// @dev 1 GovToken = 2 ETH
    purchaseTokenAddresses[0] = ETHEREUM_ADDRESS;
    exchangeRates[0] = 2 * PRECISION;

    uint256 totalTokenProvided = 100 ether;

    ITokenSaleProposal.TierInitParams memory _tierInitParams = ITokenSaleProposal
        .TierInitParams({
            metadata: ITokenSaleProposal.TierMetadata({name: "", description: ""}),
            totalTokenProvided: totalTokenProvided,
            saleStartTime: uint64(block.timestamp + 2 days),
            saleEndTime: uint64(block.timestamp + 9 days),
            claimLockDuration: 0,
            saleTokenAddress: address(govToken),
            purchaseTokenAddresses: purchaseTokenAddresses,
            exchangeRates: exchangeRates,
            minAllocationPerUser: 0,
            maxAllocationPerUser: 0,
            vestingSettings: ITokenSaleProposal.VestingSettings({
                vestingPercentage: 0,
                vestingDuration: 0,
                cliffPeriod: 0,
                unlockStep: 0
            }),
            participationDetails: new ITokenSaleProposal.ParticipationDetails[](0)
        });

    ITokenSaleProposal.TierInitParams[]
        memory tierInitParams = new ITokenSaleProposal.TierInitParams[](1);
    tierInitParams[0] = _tierInitParams;

    IGovPool.ProposalAction[] memory actionsFor = new IGovPool.ProposalAction[](2);
    actionsFor[0] = IGovPool.ProposalAction({
        executor: address(govToken),
        value: 0,
        data: abi.encodeWithSelector(
            IERC20.approve.selector,
            address(tokenSaleProposal),
            totalTokenProvided
        )
    });
    actionsFor[1] = IGovPool.ProposalAction({
        executor: address(tokenSaleProposal),
        value: 0,
        data: abi.encodeWithSelector(ITokenSaleProposal.createTiers.selector, tierInitParams)
    });

    IGovPool.ProposalAction[] memory actionsAgainst = new IGovPool.ProposalAction[](0);

    govPool.createProposal("Create tier", actionsFor, actionsAgainst);
}
```

In this tier, all optional parameters are configured with zero values. In addition, your proposal has the flexibility to combine different types of whitelists within the `participationDetails` parameter, allowing you to restrict certain addresses from participating in a tier, or to set `vestingSettings` to provide a gradual withdrawal of tokens for customers, etc.

:warning: _The `claimLockDuration` should be less than or equal to the `cliffPeriod`._

Once the `createTier` proposal is successfully executed, users are given the ability to purchase tokens in this tier by calling the `buy` method on the `TokenSaleProposal` contract. After the `saleEndTime`, they should also use the `claim` and `vestingWithdraw` methods to withdraw purchased tokens.

The proposals in the `GovPool` also allow you to call several useful methods on the `TokenSaleProposal` contract. For instance, you can use `offTiers` to halt sales in certain tiers, `addToWhitelist` if `participationDetails` contains `ParticipationType.Whitelist`, and `recover` to transfer unsold tokens back to the treasury.

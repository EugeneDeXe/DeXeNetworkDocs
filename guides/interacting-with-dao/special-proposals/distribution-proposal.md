# Distribution proposal

You can create a special proposal called a distribution proposal that distributes tokens proportionally based on the personal votes casted by users in this proposal. To create it, you need to set the main executor to the `DistributionProposal` contract, which is unique for each DAO and can be either predicted by calling the `PoolFactory`'s method or obtained from a third-party source. Now let's create a proposal that distributes 10 ETH.

```solidity
function createDistributionProposal(
    IGovPool govPool,
    IDistributionProposal distributionProposal
) external {
    uint256 proposalId = GovPool(payable(address(govPool))).latestProposalId();
    uint256 amount = 10 ether;

    IGovPool.ProposalAction[] memory actionsFor = new IGovPool.ProposalAction[](1);
    actionsFor[0] = IGovPool.ProposalAction({
        executor: address(distributionProposal),
        value: 0,
        data: abi.encodeWithSelector(
            IDistributionProposal.execute.selector,
            proposalId,
            ETHEREUM_ADDRESS,
            amount
        )
    });

    IGovPool.ProposalAction[] memory actionsAgainst = new IGovPool.ProposalAction[](0);

    govPool.createProposal("Distribution proposal", actionsFor, actionsAgainst);
}
```

:warning: _Votes against the distribution proposal are subtracted from the overall pool, and voters who do so are not eligible for rewards._

Please take note that you need to manually pass the ID of the proposal you create as a parameter to the `execute` function. If an incorrect ID is passed, the creation will be reverted, as the `DistributionProposal` contract implements the `IProposalValidator.validate` hook.

```solidity
contract DistributionProposal is IProposalValidator, /* ... */ {
    /// ...
    
    function validate(
        IGovPool.ProposalAction[] calldata actions
    ) external view override returns (bool valid) {
        uint256 proposalId = uint256(bytes32(actions[actions.length - 1].data[4:36]));

        return proposalId == GovPool(payable(govAddress)).latestProposalId();
    }
}
```

Once the distribution proposal has been executed, voters can claim their rewards by calling the `claim` method on the `DistributionProposal` contract.

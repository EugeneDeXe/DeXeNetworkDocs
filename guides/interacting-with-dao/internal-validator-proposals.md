# Internal validator proposals

There are four special internal validator proposals responsible for managing the validator-related storage: `ChangeSettings`, `ChangeBalances`, `MonthlyWithdraw`, and `OffchainProposal`. These proposals have the privilege to skip the first voting stage on the `GovPool` contract and can be directly created on the `GovValidators` contract. Take a look at the example below, where we create an internal proposal to modify `GovValidatorsToken` balances.

```solidity
function validatorsInternalProposals(IGovPool govPool) external {
    (, , address govValidatorsAddress, , ) = govPool.getHelperContracts();
    IGovValidators govValidators = IGovValidators(govValidatorsAddress);

    uint256[] memory balances = new uint256[](2);
    balances[0] = 10 ether;
    balances[1] = 20 ether;

    address[] memory validators = new address[](2);
    validators[0] = address(1);
    validators[1] = address(2);

    govValidators.createInternalProposal(
        IGovValidators.ProposalType.ChangeBalances,
        "Change Validators balances",
        abi.encodeWithSelector(IGovValidators.changeBalances.selector, balances, validators)
    );

    uint256 proposalId = GovValidators(payable(govValidatorsAddress))
        .latestInternalProposalId();

    // ...
}
```

Once you have the internal proposal ID, you can invoke the `voteInternalProposal`, `executeInternalProposal`, and `getProposalState` methods on the `GovValidators` contract, similar to what was done on the `GovPool` contract.

# Metagovernance

Consider a scenario in which we have some master organization with its own DAO, further subdivided into several slave organizations, each equipped with an independent DAO. In this setup, only the slave DAOs have the ability to vote in the master one. Such a hierarchical structure is called metagovernance and can be established using our protocol.

To make this happen, metagovernance proposals allow you to vote in one DAO on behalf of another. It's achievable by the limiting the executor to one of the deployed DAO pools, specifically the master DAO. The data included is restricted to the deposit and approval of funds, and the last action is exclusively a vote for the proposal in the master DAO (in the `actionsFor` list) and a vote against it (in the `actionsAgainst` list).

It's important to note that the `actionsAgainst` parameter is only non-empty in metagovernance proposals. The `GovPool` contract enforces a validation process to ensure that proposals specifying `actionsAgainst` adhere to the established criteria for metagovernance proposals. Specifically, `actionsFor` and `actionsAgainst` should have the same length and parameters, except for the last one, where the `isVoteFor` parameter differs.

Apart from ordinary proposals, metagovernance proposals can be executed if the quorum against is reached. In such cases, the proposal status will change to `ExecutedAgainst`, and all users who voted against will receive rewards.

Here's an example of creating a proposal that votes in the master DAO on behalf of the slave DAO.

```solidity
function metagovernanceProposal(IGovPool masterGovPool, IGovPool slaveGovPool) external {
    (, address masterUserKeeperAddress, , , ) = masterGovPool.getHelperContracts();
    address tokenAddress = IGovUserKeeper(masterUserKeeperAddress).tokenAddress();

    require(tokenAddress != address(0), "Zero token");

    uint256 proposalId = 1;
    uint256 amount = 10 ether;
    uint256[] memory nftIds = new uint256[](0);

    require(
        IERC20(tokenAddress).balanceOf(address(slaveGovPool)) >= amount,
        "Insufficient balance"
    );
    require(
        masterGovPool.getProposalState(proposalId) == IGovPool.ProposalState.Voting,
        "Not voting state"
    );

    IGovPool.ProposalAction[] memory actionsFor = new IGovPool.ProposalAction[](3);
    IGovPool.ProposalAction[] memory actionsAgainst = new IGovPool.ProposalAction[](3);

    actionsFor[0] = actionsAgainst[0] = IGovPool.ProposalAction({
        executor: tokenAddress,
        value: 0,
        data: abi.encodeWithSelector(IERC20.approve.selector, masterUserKeeperAddress, amount)
    });
    actionsFor[1] = actionsAgainst[1] = IGovPool.ProposalAction({
        executor: address(masterGovPool),
        value: 0,
        data: abi.encodeWithSelector(IGovPool.deposit.selector, amount, nftIds)
    });

    /// @dev vote for the proposal in the master DAO
    actionsFor[2] = IGovPool.ProposalAction({
        executor: address(masterGovPool),
        value: 0,
        data: abi.encodeWithSelector(IGovPool.vote.selector, proposalId, true, amount, nftIds)
    });

    /// @dev vote against the proposal in the master DAO
    actionsAgainst[2] = IGovPool.ProposalAction({
        executor: address(masterGovPool),
        value: 0,
        data: abi.encodeWithSelector(IGovPool.vote.selector, proposalId, false, amount, nftIds)
    });

    slaveGovPool.createProposal("Metagovernance proposal", actionsFor, actionsAgainst);
}
```

# Rewards

For most actions in the DAO, users receive rewards that are categorized into static and voting ones. Upon receiving a static reward, users can claim it definitively if the proposal is successfully executed. One such static reward is the reward for the proposal creation, which is determined by the `creationReward` parameter. When a user `moveProposalToValidators` or `execute` a proposal, they earn another static reward set by the `executionReward` parameter. In contrast, voting rewards are not fixed. The protocol allows users to `cancelVote` or `undelegate` assets, which may result in the "burning" of potential rewards.

It's important to note that micropool rewards bear a resemblance to voting rewards, although they can be retrieved and claimed through different methods. When you delegate your assets to someone, it involves a complex staking-like mechanism. For example, with multiple delegators and one delegatee, when the delegatee votes, all delegators' votes are automatically aggregated with his own. The delegatee then receives rewards for the delegated power of all delegators. A certain percentage is claimed by the delegatee, while the remaining rewards are distributed proportionally among the delegators, as in the common staking.

Consider three proposals have been executed within our DAO. Preceding these events, we delegated assets to `address(1)` and actively engaged in these proposals using methods like `createProposal`, `moveProposalToValidators`, `execute`, and `vote`. Furthermore, our delegatee voted in these proposals as well, entitling us to a share of the rewards. Now, our objective is to determine our rewards.&#x20;

```solidity
function getRewards(IGovPool govPool) external view {
    address user = address(this);
    address delegatee = address(1);
    uint256[] memory proposalIds = new uint256[](3);
    proposalIds[0] = 1;
    proposalIds[1] = 2;
    proposalIds[2] = 3;

    IGovPool.PendingRewardsView memory pendingRewards = govPool.getPendingRewards(
        user,
        proposalIds
    );

    IGovPool.DelegatorRewards memory delegatorRewards = govPool.getDelegatorRewards(
        proposalIds,
        user,
        delegatee
    );

    /// ...
}
```

After obtaining the rewards, you can withdraw them using the corresponding claim methods.

```solidity
function claimRewards(IGovPool govPool) external {
    address user = address(this);
    address delegatee = address(1);
    uint256[] memory proposalIds = new uint256[](3);
    proposalIds[0] = 1;
    proposalIds[1] = 2;
    proposalIds[2] = 3;

    govPool.claimRewards(proposalIds, user);
    govPool.claimMicropoolRewards(proposalIds, user, delegatee);
}
```

# Delegations

#### Regular Delegations

If you prefer not to participate directly in the voting process but trust a specific individual within the DAO, you have the option to delegate your funds to them. By doing so, the tokens you delegate are added to the total voting power of the delegatee (a.k.a. micropool). When the proposal that the delegatee voted on is executed, a specific percentage of rewards is directed to the delegatee, while the remaining portion is distributed proportionally among all delegators. This distribution of rewards takes into account the delegated amounts, ensuring a fair distribution among delegators based on their delegation amounts. When you delegate your assets, the delegatee's voting power increases in active proposals. This means there will be a revote that takes your delegated assets into consideration.&#x20;

It's important to note that users can only vote using their entire delegated assets. Even if you vote with personal assets, the assets delegated to you will be automatically added to your votes.  If the `delegatedVotingAllowed` parameter in proposal settings is set to false, then delegated to you assets won't be added to your vote. However, you will be able to vote with the assets you have delegated to someone else.&#x20;

Below is an example illustrating how to delegate all available assets.&#x20;

```solidity
function delegate(IGovPool govPool) external {
    (uint256 amount, uint256[] memory nftIds) = govPool.getWithdrawableAssets(address(this));
    address delegatee = address(1);
    
    govPool.delegate(delegatee, amount, nftIds);
}
```

You also have the option to undelegate assets at any time. In this scenario, the delegatee will automatically revote in active proposals, excluding your assets.

```solidity
function undelegate(IGovPool govPool) external {
    (, address govUserKeeperAddress, , , ) = govPool.getHelperContracts();
    IGovUserKeeper govUserKeeper = IGovUserKeeper(govUserKeeperAddress);

    (, IGovUserKeeper.DelegationInfoView[] memory delegationsInfo) = govUserKeeper.delegations(
        address(this),
        false
    );

    for (uint256 i = 0; i < delegationsInfos.length; ++i) {
        govPool.undelegate(
            delegationsInfos[i].delegatee,
            delegationsInfos[i].delegatedTokens,
            delegationsInfos[i].delegatedNfts
        );
    }
}
```

:warning: _Delegate and undelegate cannot be made within the same block (via multicall). This is done to prevent possible flash-loan attacks._

#### Treasury Delegations

Additionally, you can request delegation directly from the `GovPool` contract's balance, which is commonly known as the treasury. This can be done through the voting process (proposal). Treasury delegations work similarly to standard delegations except that they can only be issued to experts (`ERC721Expert` holders). When treasury is delegated or undelegated to you, its entire amount will be automatically added or removed from all your votes in active proposals. Moreover, you will receive a certain percentage of rewards if proposal is successfully executed. Here are some examples of creating proposals for treasury delegation and undelegation. We'll cover voting and executing proposals in the next chapter about the proposal life cycle. Please refer to the example below for further clarification.

<pre class="language-solidity"><code class="lang-solidity">function delegateTreasury(IGovPool govPool) external {
    (, address govUserKeeperAddress, , , ) = govPool.getHelperContracts();
    IGovUserKeeper govUserKeeper = IGovUserKeeper(govUserKeeperAddress);

    address delegatee = address(1);
    uint256 amount = 1 ether;
    uint256[] memory nftIds = new uint256[](1);
    nftIds[0] = 1;

<strong>    require(govPool.getExpertStatus(delegatee), "Not an expert");
</strong>    require(
        IERC20(govUserKeeper.tokenAddress()).balanceOf(address(govPool)) >= amount,
        "GovPool has insufficient ERC20 balance"
    );
    require(
        IERC721(govUserKeeper.nftAddress()).ownerOf(nftIds[0]) == address(govPool),
        "GovPool has insufficient ERC721 balance"
    );

    IGovPool.ProposalAction[] memory actionsFor = new IGovPool.ProposalAction[](1);
    actionsFor[0] = IGovPool.ProposalAction({
        executor: address(govPool),
        value: 0,
        data: abi.encodeWithSelector(
            IGovPool.delegateTreasury.selector,
            delegatee,
            amount,
            nftIds
        )
    });

    IGovPool.ProposalAction[] memory actionsAgainst = new IGovPool.ProposalAction[](0);

    govPool.createProposal("DelegateTreasury", actionsFor, actionsAgainst);
}

function undelegateTreasury(IGovPool govPool) external {
    address delegatee = address(1);
    uint256 amount = 1 ether;
    uint256[] memory nftIds = new uint256[](1);
    nftIds[0] = 1;

    IGovPool.ProposalAction[] memory actionsFor = new IGovPool.ProposalAction[](1);
    actionsFor[0] = IGovPool.ProposalAction({
        executor: address(govPool),
        value: 0,
        data: abi.encodeWithSelector(
            IGovPool.undelegateTreasury.selector,
            delegatee,
            amount,
            nftIds
        )
    });

    IGovPool.ProposalAction[] memory actionsAgainst = new IGovPool.ProposalAction[](0);

    govPool.createProposal("UndelegateTreasury", actionsFor, actionsAgainst);
}
</code></pre>

:warning: _Treasury balance won't be added to your vote in_ `delegateTreasury` _and_ `undelegateTreasury` _proposals concerning you as a delegatee._

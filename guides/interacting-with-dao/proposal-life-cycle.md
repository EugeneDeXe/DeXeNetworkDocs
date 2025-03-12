# Proposal life cycle

In the proposal life cycle, the initial step is its creation. We have previously discussed the creation of proposals that alter the state of system contracts, but it's important to note that the executor can be any contract. Suppose we have the `UnknownToken` token, and ownership of this token has been transferred to the DAO.

<pre class="language-solidity"><code class="lang-solidity"><strong>contract UnknownToken is ERC20, Ownable {
</strong>    constructor(string memory name, string memory symbol) ERC20(name, symbol) {}

    function mint(address to, uint256 amount) external onlyOwner {
        _mint(to, amount);
    }
}
</code></pre>

Now, let's consider a scenario where we want to execute the `mint` function of this token via a proposal.

```solidity
function createProposal(GovPool govPool, IERC20 token) external returns (uint256 proposalId) {
    address receiver = address(this);
    uint256 amount = 1 ether;

    IGovPool.ProposalAction[] memory actionsFor = new IGovPool.ProposalAction[](1);
    actionsFor[0] = IGovPool.ProposalAction({
        executor: address(token),
        value: 0,
        data: abi.encodeWithSelector(UnknownToken.mint.selector, receiver, amount)
    });

    IGovPool.ProposalAction[] memory actionsAgainst = new IGovPool.ProposalAction[](0);

    govPool.createProposal("Mint token", actionsFor, actionsAgainst);

    return GovPool(govPool).latestProposalId();
}
```

You can also add specific validations that will be passed each time you create a proposal with your contract functioning as the main executor. To achieve this, you should implement the `IProposalValidator` interface. In this particular case, the validation ensures that the only action within the proposal is the minting of some amount of tokens. If the `validate` method returns false, the creation of the proposal will be reverted.

```solidity
contract UnknownToken is IProposalValidator {
    function validate(
        IGovPool.ProposalAction[] calldata actions
    ) external view override returns (bool valid) {
        return actions.length == 1 && bytes4(actions[0].data[0:4]) == UnknownToken.mint.selector;
    }
}
```

Once created, a proposal initially enters the `Voting` state, making it available for user to cast their votes. The voting period remains open as long as the quorum is not reached or the voting time hasn't elapsed. Users may also vote against proposals. If votes against the proposal reach a quorum, the status of the proposal becomes `Defeated`. Now we can pass the `proposalId` from the just-created proposal to the voting function and cast a vote in favor of it.

```solidity
function vote(IGovPool govPool, uint256 proposalId) external {
    require(
        govPool.getProposalState(proposalId) == IGovPool.ProposalState.Voting,
        "Not voting state"
    );

    bool isVoteFor = true;
    uint256 amount = 1 ether;
    uint256[] memory nftIds = new uint256[](1);

    govPool.vote(proposalId, isVoteFor, amount, nftIds);
}
```

To save on gas, consider utilizing the `createProposalAndVote` function.

```solidity
function createProposalAndVote(IGovPool govPool) external {
    // ...
    
    govPool.createProposalAndVote("MintToken", actionsFor, actionsAgainst, amount, nftIds);
}
```

It's possible to cancel your entire vote on the proposal, including the delegated power, by simply calling the `cancelVote` method. After that, your assets will no longer be locked and you can withdraw them.

```solidity
function cancelVote(IGovPool govPool, uint256 proposalId) external {
    require(
        govPool.getProposalState(proposalId) == IGovPool.ProposalState.Voting,
        "Not voting state"
    );

    govPool.cancelVote(proposalId);
}
```

Assuming our proposal has reached the quorum and has exceeded `duration` if `earlyCompletion` is false, it can exist in one of three possible states. The first state is `Locked` in which case we only need to wait for the `executionDelay`. The second state is `SucceededFor` or `SucceededAgainst` indicating that it is ready for execution. In this case, we can simply call the `execute` method.

```solidity
function execute(IGovPool govPool, uint256 proposalId) external {
    IGovPool.ProposalState proposalState = govPool.getProposalState(proposalId);

    require(
        proposalState == IGovPool.ProposalState.SucceededFor ||
            proposalState == IGovPool.ProposalState.SucceededAgainst,
        "Not succeeded state"
    );

    govPool.execute(proposalId);
}
```

:warning: _SucceededAgainst status is reachable only in meta-governance proposals not covered in this chapter._

The `WaitingForVotingTransfer` state, indicating that `validatorsVote` is true, signifies that the proposal is ready to be moved to the second stage of voting. To initiate this, the `moveProposalToValidators` method should be called by any user. This method triggers the creation of the corresponding external proposal in the `GovValidators` contract. External proposals for the `GovValidators` contract have the same ids as those on the `GovPool` contract.

```solidity
function moveProposalToValidators(IGovPool govPool, uint256 proposalId) external {
    require(
        govPool.getProposalState(proposalId) ==
            IGovPool.ProposalState.WaitingForVotingTransfer,
        "Not waiting for transfer state"
    );

    govPool.moveProposalToValidators(proposalId);
}
```

Following the transition of the proposal to the validators, the validators will have the ability to cast their votes for it.

```solidity
function validatorVote(IGovPool govPool, uint256 proposalId) external {
    (, , address govValidatorsAddress, , ) = govPool.getHelperContracts();
    IGovValidators govValidators = IGovValidators(govValidatorsAddress);

    bool isInternal = false;
    require(
        govValidators.getProposalState(proposalId, isInternal) ==
            IGovValidators.ProposalState.Voting,
        "Not voting state"
    );

    uint256 amount = 1 ether;
    bool isVoteFor = true;

    govValidators.voteExternalProposal(proposalId, amount, isVoteFor);
}
```

Once the external proposal has the `Succeeded` status on the `GovValidators` contract, it will also attain the `SucceededFor` or `SucceededAgainst` status on the `GovPool` contract, allowing it to be executed in the usual manner.

```solidity
function executeAfterValidators(IGovPool govPool, uint256 proposalId) external {
    (, , address govValidatorsAddress, , ) = govPool.getHelperContracts();
    IGovValidators govValidators = IGovValidators(govValidatorsAddress);

    bool isInternal = false;
    require(
        govValidators.getProposalState(proposalId, isInternal) ==
            IGovValidators.ProposalState.Succeeded,
        "Not succeeded state"
    );

    IGovPool.ProposalState proposalState = govPool.getProposalState(proposalId);

    assert(
        proposalState == IGovPool.ProposalState.SucceededFor ||
            proposalState == IGovPool.ProposalState.SucceededAgainst
    );

    govPool.execute(proposalId);
}
```

Well done! When you call the `execute` method, the tokens are minted to the address specified in the proposal. You can verify this by simply calling the `balanceOf` method on the token contract.

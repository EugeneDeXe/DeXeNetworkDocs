# VotePower

When casting your vote, all the tokens you voted with are combined into a singular voting power, possibly adjusted by the `VotePower` contract. By default, it does nothing or uses a polynomial formula that takes into account the user's expert status and treasury balance.

Suppose we aim to customize this contract's logic: if a user is on the whitelist, we will square its vote power; otherwise, it remains unchanged. In the example below, you'll find such a contract with two methods managing the whitelist, while other methods implement the `IVotePower` interface. Currently, there is no need to modify the`transformVotesFull` and `getVotesRatio` methods. The `transformVotes` function is responsible for the vote power adjustment.

```solidity
contract SquareWhitelistPower is IVotePower, OwnableUpgradeable {
    mapping(address => bool) public whitelist;

    function __SquareWhitelistPower_init() external initializer {
        __Ownable_init();
    }

    function transformVotes(
        address user,
        uint256 votes
    ) public view override returns (uint256 resultingVotes) {
        return whitelist[user] ? votes * votes : votes;
    }

    function transformVotesFull(
        address user,
        uint256 votes,
        uint256,
        uint256,
        uint256
    ) external view override returns (uint256 resultingVotes) {
        return transformVotes(user, votes);
    }

    function addToWhitelist(address user) external onlyOwner {
        whitelist[user] = true;
    }

    function removeFromWhitelist(address user) external onlyOwner {
        delete whitelist[user];
    }

    function getVotesRatio(address) external pure override returns (uint256 votesRatio) {
        return PRECISION;
    }
}
```

Similar to the `ERC721Power`, you need to deploy and initialize your custom `VotePower` contract on your own. Then, there are two ways to integrate it into the DAO. The first method is to pass its address along with the `poolParameters` during the deployment of your DAO.

```solidity
function createDAO(IVotePower votePower) external {
    IPoolFactory.GovPoolDeployParams memory poolParameters = _getPoolParameters();

    poolParameters.votePowerParams = IPoolFactory.VotePowerDeployParams({
        voteType: IPoolFactory.VotePowerType.CUSTOM_VOTES,
        initData: "",
        presetAddress: address(votePower)
    });

    poolFactory.deployGovPool(poolParameters);
}
```

The other way is to change `VotePower` via the voting. Unlike `ERC721Power`, `VotePower` can be modified multiple times.

```solidity
function setVotePower(IVotePower votePower) external {
    IGovPool.ProposalAction[] memory actionsFor = new IGovPool.ProposalAction[](1);
    actionsFor[0] = IGovPool.ProposalAction({
        executor: address(govPool),
        value: 0,
        data: abi.encodeWithSelector(IGovPool.changeVotePower.selector, votePower)
    });
    
    IGovPool.ProposalAction[] memory actionsAgainst = new IGovPool.ProposalAction[](0);

    govPool.createProposal("Set VotePower", actionsFor, actionsAgainst);
}
```

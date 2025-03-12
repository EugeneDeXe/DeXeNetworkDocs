# Deploying DAO

This page explains how to deploy your own DAO pool and retrieve its on-chain address for further interactions.

First, find the `PoolFactory` contract address on the chain you are using. Once found, simply call the `PoolFactory.deployGovPool` method with the appropriate parameters.

<pre class="language-solidity"><code class="lang-solidity"><strong>contract CreateDAO {
</strong>    IPoolFactory public immutable poolFactory;

    // ...

    constructor(address _poolFactory) {
        poolFactory = IPoolFactory(_poolFactory);
    }

    function createDAO() external {
        IPoolFactory.GovPoolDeployParams memory poolParameters = _getPoolParameters();

        poolFactory.deployGovPool(poolParameters);

        // ...
    }

    function _getPoolParameters()
        internal
        pure
        returns (IPoolFactory.GovPoolDeployParams memory poolParameters)
    {
        // ...
    }
}

</code></pre>

The factory under the hood uses the create2 mechanism to predict the pool address. It can be predicted either before or after the deployment itself, using the deployer's address `tx.origin` and the pool name `poolParameters.name`. With this knowledge, let's calculate the deployed GovPool address and, to see if it works, get the `GovUserKeeper` contract by calling the `GovPool.getHelperContracts` method.

{% code fullWidth="false" %}
```solidity
contract CreateDAO {
    IPoolFactory public immutable poolFactory;

    IGovPool public govPool;
    IGovUserKeeper public govUserKeeper;

    constructor(address _poolFactory) {
        poolFactory = IPoolFactory(_poolFactory);
    }

    function createDAO() external {
        IPoolFactory.GovPoolDeployParams memory poolParameters = _getPoolParameters();

        poolFactory.deployGovPool(poolParameters);

        IPoolFactory.GovPoolPredictedAddresses memory predictedAddresses = poolFactory
            .predictGovAddresses(tx.origin, poolParameters.name);

        govPool = IGovPool(predictedAddresses.govPool);

        (, address govUserKeeperAddress, , , ) = govPool.getHelperContracts();
        govUserKeeper = IGovUserKeeper(govUserKeeperAddress);
    }

    function _getPoolParameters()
        internal
        pure
        returns (IPoolFactory.GovPoolDeployParams memory poolParameters)
    {
        // ...
    }
}
```
{% endcode %}

Now, the only task remaining is to implement the `_getPoolParameters` method. This task is not trivial, so let's break it down step by step. Initially, we'll configure the proposal settings. Assuming our DAO has no validator voting, the reward token is the native currency, and proposals require 25% of the total votes for execution, the following settings should be configured. It's worth noting that constants from the listing can be imported from `core/Globals.sol`.

```solidity
contract CreateDAO {
    uint256 constant PERCENTAGE_100 = 10 ** 27;
    uint256 constant PRECISION = 10 ** 25;
    address constant ETHEREUM_ADDRESS = 0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE;
    
    // ...
    
    function _getProposalSettings() internal pure returns (IGovSettings.ProposalSettings memory) {
        return
            IGovSettings.ProposalSettings({
                earlyCompletion: true,
                delegatedVotingAllowed: true,
                validatorsVote: false,
                duration: 7 days,
                durationValidators: 7 days,
                executionDelay: 0,
                quorum: uint128(PERCENTAGE_100 / 4),
                quorumValidators: 0,
                minVotesForVoting: 0,
                minVotesForCreating: 0,
                rewardsInfo: IGovSettings.RewardsInfo({
                    rewardToken: ETHEREUM_ADDRESS,
                    creationReward: 0.005 ether,
                    executionReward: 0.001 ether,
                    voteRewardsCoefficient: PRECISION
                }),
                executorDescription: ""
            });
    }
}
```

To configure `GovSettings`, it is necessary to match the executors to their respective settings. In this context, an executor refers to a contract called during the execution of a proposal, and each executor can have its own settings. The protocol will choose the appropriate settings based on the executors associated with the proposal. If there is more than one executor, the settings of the main executor will be considered (the last one in the list).

By default, there are three types of executor settings that must be defined: `DEFAULT` (for executors without specific settings), `INTERNAL` (for contracts such as `GovPool`, `GovSettings`, `GovUserKeeper`), and `VALIDATORS` (for the `GovValidators` contract). Additionally, special settings are set for an executor at the `address(1337)`, which can represent any desired contract. For example, if the main executor is set to `address(1337)`, a 10% quorum validator voting will be activated, unlike other executors.

```solidity
function _getSettingsParams()
    internal
    pure
    returns (IPoolFactory.SettingsDeployParams memory)
{
    IGovSettings.ProposalSettings[]
        memory proposalSettings = new IGovSettings.ProposalSettings[](4);

    proposalSettings[uint256(IGovSettings.ExecutorType.DEFAULT)] = _getProposalSettings();
    proposalSettings[uint256(IGovSettings.ExecutorType.INTERNAL)] = _getProposalSettings();
    proposalSettings[uint256(IGovSettings.ExecutorType.VALIDATORS)] = _getProposalSettings();

    proposalSettings[3] = _getProposalSettings();
    proposalSettings[3].validatorsVote = true;
    proposalSettings[3].quorumValidators = uint128(PERCENTAGE_100 / 10);

    address[] memory additionalProposalExecutors = new address[](1);
    additionalProposalExecutors[0] = address(1337);

    return IPoolFactory.SettingsDeployParams(proposalSettings, additionalProposalExecutors);
}
```

Setting up `GovUserKeeper` is straightforward. We just need to specify the tokens with which users will vote (use the `address(0)` if no token is required). If the NFT is used, it should support the [ERC165 standard](https://eips.ethereum.org/EIPS/eip-165), and individual voting power should be set unless it supports `IERC721Power`. If the NFT doesn't support `IERC721Enumerable`, the total supply of NFTs should be manually specified. Note that at least one token (`tokenAddress` or `nftAddress`) must be set. Suppose that in the listing below, `address(2)` is the address of the `IERC721Enumerable`.

```solidity
function _getUserKeeper() internal pure returns (IPoolFactory.UserKeeperDeployParams memory) {
    return
        IPoolFactory.UserKeeperDeployParams({
            tokenAddress: address(1),
            nftAddress: address(2),
            individualPower: 10 ** 18,
            nftsTotalSupply: 0
        });
}
```

Validators vote on two types of proposals within the `GovValidators` contract. The first type, external proposals, are moved from the `GovPool` to the second level of voting. The second type, internal proposals, involve changes to the `GovValidators` state and require special `proposalSettings`. Validators use a special `ERC20Snapshot` token that is deployed with the pool. Therefore, it is important to set the initial validator addresses and their balances to be minted.

```solidity
function _getValidatorsParams()
    internal
    pure
    returns (IPoolFactory.ValidatorsDeployParams memory)
{
    address[] memory validators = new address[](2);
    validators[0] = address(0xaa);
    validators[1] = address(0xbb);

    uint256[] memory balances = new uint256[](2);
    balances[0] = 1 ether;
    balances[1] = 2 ether;

    return
        IPoolFactory.ValidatorsDeployParams({
            name: "GovValidatorToken",
            symbol: "GVT",
            proposalSettings: IGovValidators.ProposalSettings({
                duration: 7 days,
                executionDelay: 0,
                quorum: uint128(PERCENTAGE_100 / 10)
            }),
            validators: validators,
            balances: balances
        });
}
```

The `ERC20Gov` token is also deployed with the pool and serves as a token that can be minted, burned, paused, and has a cap. Its management is handled by the DAO. Initial holders can be established. The difference between `mintedTotal` and `amounts` will be sent to the DAO. This token can also be sold in the `TokenSaleProposal`.

```solidity
function _getGovTokenParams() internal pure returns (IERC20Gov.ConstructorParams memory) {
    return
        IERC20Gov.ConstructorParams({
            name: "GovToken",
            symbol: "GT",
            users: new address[](0),
            cap: 100 ether,
            mintedTotal: 10 ether,
            amounts: new uint256[](0)
        });
}
```

The user votes can be transformed based on specific rules set by the `VotePower` contract. There are two options available for deployment:

1. Deploy our `VotePower` contract, supporting one of two types of voting: `LINEAR_VOTES`, where the vote power equals the exact number of tokens, and `POLYNOMIAL_VOTES`, where the vote power is calculated using a polynomial formula. In this case, `initData` is expected to be passed.
2. Customize and deploy your own `VotePower` contract, then pass the `presetAddress`.

```solidity
function _getVotePowerParams()
    internal
    pure
    returns (IPoolFactory.VotePowerDeployParams memory)
{
    return
        IPoolFactory.VotePowerDeployParams({
            voteType: IPoolFactory.VotePowerType.LINEAR_VOTES,
            initData: abi.encodeWithSelector(LinearPower.__LinearPower_init.selector),
            presetAddress: address(0)
        });
}
```

Let's finally consolidate all the parameters and implement the `_getPoolParameters` method. The parameter `onlyBABTHolders` indicates that only holders of Binance's SBT token can participate in the DAO.

```solidity
function _getPoolParameters()
    internal
    pure
    returns (IPoolFactory.GovPoolDeployParams memory poolParameters)
{
    return
        IPoolFactory.GovPoolDeployParams({
            settingsParams: _getSettingsParams(),
            validatorsParams: _getValidatorsParams(),
            userKeeperParams: _getUserKeeperParams(),
            tokenParams: _getGovTokenParams(),
            votePowerParams: _getVotePowerParams(),
            verifier: address(0),
            onlyBABTHolders: true,
            descriptionURL: "",
            name: "ExampleDAO"
        });
}
```

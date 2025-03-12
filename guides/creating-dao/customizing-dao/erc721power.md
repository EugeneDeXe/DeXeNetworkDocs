# ERC721Power

The `ERC721Power` contract is designed to enhance the functionalities of `ERC721` through the introduction of a dynamic NFT power mechanism based on collateralization and time. This token can be selected for voting in proposals over the typical `ERC721/ERC721Enumerable` that have the same power for each NFT called `individualPower`. The protocol provides two `ERC721Power` contracts: `ERC721EquivalentPower`, in which the `nftPower` is determined by multiplying `powerEquivalent` with the ratio of `nftRawPower` to the `totalRawPower`; and `ERC721RawPower`, where the power of NFTs corresponds directly to their individual `nftRawPower`.

It's important to note that this contract is not deployed by the factory, so you must either have the existent contract address or deploy it independently. There are two ways to integrate it into the DAO. The simpler approach involves passing it along with the `poolParameters` during the deployment of your DAO.

```solidity
function createDAO(IERC721Power nftPower) external {
    IPoolFactory.GovPoolDeployParams memory poolParameters = _getPoolParameters();

    require(nftPower.supportsInterface(type(IERC721Power).interfaceId), "Not a ERC721Power");

    poolParameters.userKeeperParams.nftAddress = address(nftPower);

    poolFactory.deployGovPool(poolParameters);
}
```

An alternative way for configuring the custom NFT address is to deploy a pool with a zero `nftAddress` and then initiate a proposal for the change.

```solidity
function setERC721Address(IERC721Power nftPower) external {
    require(nftPower.supportsInterface(type(IERC721Power).interfaceId), "Not a ERC721Power");

    IGovPool.ProposalAction[] memory actionsFor = new IGovPool.ProposalAction[](1);
    actionsFor[0] = IGovPool.ProposalAction({
        executor: address(govUserKeeper),
        value: 0,
        data: abi.encodeWithSelector(IGovUserKeeper.setERC721Address.selector, nftPower)
    });
    
    IGovPool.ProposalAction[] memory actionsAgainst = new IGovPool.ProposalAction[](0);

    govPool.createProposal("Set ERC721Power", actionsFor, actionsAgainst);
}
```

:warning: _Once the `nftAddress` is set during pool deployment or by the voting, it becomes unchangeable._

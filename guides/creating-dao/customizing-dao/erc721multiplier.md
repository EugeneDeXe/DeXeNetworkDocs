# ERC721Multiplier

With the exception of the `VotePower` contract, the `ERC721Multiplier` does not increase voting power, but rather directly boosts your rewards. The `ERC721Multiplier` contract allows users to lock and unlock their NFTs, each of which is associated with a specific duration and multiplier, much like a coupon. If a user locks an NFT before claiming rewards, they will receive additional rewards based on the multiplier of the locked NFT.

Not all DAOs follow the same logic for `ERC721Multiplier`. In the case of DEXE DAO, it employs a custom contract called `DexeERC721Multiplier`. This implementation reduces the multiplier by 10% each time the token is unlocked. Additionally, it considers the average user balance when calculating extra rewards. You can deploy your own version of `ERC721Multiplier` by implementing the `IAbstractERC721Multiplier` interface. Following that, you can create a proposal to change the `ERC721Multiplier` in the DAO as in the example below.

```solidity
function setNftMultiplierAddress(IAbstractERC721Multiplier nftMultiplier) external {
    require(
        nftMultiplier.supportsInterface(type(IAbstractERC721Multiplier).interfaceId),
        "Not a ERC721Multiplier"
    );

    IGovPool.ProposalAction[] memory actionsFor = new IGovPool.ProposalAction[](1);
    actionsFor[0] = IGovPool.ProposalAction({
        executor: address(govPool),
        value: 0,
        data: abi.encodeWithSelector(IGovPool.setNftMultiplierAddress.selector, nftMultiplier)
    });

    IGovPool.ProposalAction[] memory actionsAgainst = new IGovPool.ProposalAction[](0);

    govPool.createProposal("Set ERC721Multiplier", actionsFor, actionsAgainst);
}
```

# Table of contents

## getting started

* [Architecture Overview](README.md)
* [Glossary](getting-started/glossary.md)

## guides

* [Creating DAO](guides/creating-dao/README.md)
  * [Deploying DAO](guides/creating-dao/deploying-dao.md)
  * [Customizing DAO](guides/creating-dao/customizing-dao/README.md)
    * [VotePower](guides/creating-dao/customizing-dao/votepower.md)
    * [ERC721Power](guides/creating-dao/customizing-dao/erc721power.md)
    * [ERC721Multiplier](guides/creating-dao/customizing-dao/erc721multiplier.md)
* [Interacting with DAO](guides/interacting-with-dao/README.md)
  * [Deposit/Withdraw](guides/interacting-with-dao/deposit-withdraw.md)
  * [Delegations](guides/interacting-with-dao/delegations.md)
  * [Proposal life cycle](guides/interacting-with-dao/proposal-life-cycle.md)
  * [Rewards](guides/interacting-with-dao/rewards.md)
  * [Metagovernance](guides/interacting-with-dao/metagovernance.md)
  * [Internal validator proposals](guides/interacting-with-dao/internal-validator-proposals.md)
  * [Special proposals](guides/interacting-with-dao/special-proposals/README.md)
    * [Distribution proposal](guides/interacting-with-dao/special-proposals/distribution-proposal.md)
    * [Token sale proposal](guides/interacting-with-dao/special-proposals/token-sale-proposal.md)
* [Usage of subgraphs](guides/usage-of-subgraphs.md)

## contract interfaces

* [Core contracts](contract-interfaces/core-contracts/README.md)
  * [IPriceFeed](contract-interfaces/core-contracts/ipricefeed.md)
  * [IContractsRegistry](contract-interfaces/core-contracts/icontractsregistry.md)
  * [ICoreProperties](contract-interfaces/core-contracts/icoreproperties.md)
* [Factory contracts](contract-interfaces/factory-contracts/README.md)
  * [IPoolRegistry](contract-interfaces/factory-contracts/ipoolregistry.md)
  * [IPoolFactory](contract-interfaces/factory-contracts/ipoolfactory.md)
* [Gov contracts](contract-interfaces/gov-contracts/README.md)
  * [ERC20](contract-interfaces/gov-contracts/erc20/README.md)
    * [IERC20Gov](contract-interfaces/gov-contracts/erc20/ierc20gov.md)
  * [ERC721](contract-interfaces/gov-contracts/erc721/README.md)
    * [experts](contract-interfaces/gov-contracts/erc721/experts/README.md)
      * [IERC721Expert](contract-interfaces/gov-contracts/erc721/experts/ierc721expert.md)
    * [multipliers](contract-interfaces/gov-contracts/erc721/multipliers/README.md)
      * [IAbstractERC721Multiplier](contract-interfaces/gov-contracts/erc721/multipliers/iabstracterc721multiplier.md)
      * [IERC721Multiplier](contract-interfaces/gov-contracts/erc721/multipliers/ierc721multiplier.md)
      * [IDexeERC721Multiplier](contract-interfaces/gov-contracts/erc721/multipliers/idexeerc721multiplier.md)
    * [powers](contract-interfaces/gov-contracts/erc721/powers/README.md)
      * [IERC721Power](contract-interfaces/gov-contracts/erc721/powers/ierc721power.md)
  * [proposals](contract-interfaces/gov-contracts/proposals/README.md)
    * [IProposalValidator](contract-interfaces/gov-contracts/proposals/iproposalvalidator.md)
    * [IDistributionProposal](contract-interfaces/gov-contracts/proposals/idistributionproposal.md)
    * [ITokenSaleProposal](contract-interfaces/gov-contracts/proposals/itokensaleproposal.md)
  * [settings](contract-interfaces/gov-contracts/settings/README.md)
    * [IGovSettings](contract-interfaces/gov-contracts/settings/igovsettings.md)
  * [user-keeper](contract-interfaces/gov-contracts/user-keeper/README.md)
    * [IGovUserKeeper](contract-interfaces/gov-contracts/user-keeper/igovuserkeeper.md)
  * [validators](contract-interfaces/gov-contracts/validators/README.md)
    * [IGovValidators](contract-interfaces/gov-contracts/validators/igovvalidators.md)
    * [IGovValidatorsToken](contract-interfaces/gov-contracts/validators/igovvalidatorstoken.md)
  * [voting](contract-interfaces/gov-contracts/voting/README.md)
    * [IVotePower](contract-interfaces/gov-contracts/voting/ivotepower.md)
  * [IGovPool](contract-interfaces/gov-contracts/igovpool.md)

## contracts deployments

* [Prod (BSC/ETH)](contracts-deployments/prod-bsc-eth.md)
* [Stage (BSC Testnet/Sepolia)](contracts-deployments/stage-bsc-testnet-sepolia.md)

## subgraphs deployments

* [Prod (BSC)](subgraphs-deployments/prod-bsc.md)
* [Prod (ETH)](subgraphs-deployments/prod-eth.md)
* [Stage (BSC Testnet)](subgraphs-deployments/stage-bsc-testnet.md)

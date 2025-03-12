# Architecture Overview

<figure><img src=".gitbook/assets/diagram (5).svg" alt=""><figcaption></figcaption></figure>

## Introduction

DEXE contracts offer a strong and effective structure for making and communicating with DAOs. The system features multiple contracts divided into three key groups including `core`, `gov`, and `factory`.

## Components

#### Core contracts

The core contracts play a crucial role in the functionality of the platform. The `ContractsRegistry` contract acts as a central hub, keeping track of all the essential contracts, ensuring the system can be upgraded easily, and allowing for the smooth integration of necessary tools. The another core contract called `CoreProperties` is responsible for storing universal system constants that all modules comply with. When a new DAO is established, it uses these set values to guarantee consistency and reliability across the complete DEXE ecosystem.

In basic terms, the core contracts make sure that all vital rules and tools are well-kept, whether you are upgrading the system or creating a new DAO.

#### Factory contracts

The factory contracts do several important things. Firstly, the `PoolRegistry` keeps a list of all the pools that people created, and allows all of them to be upgraded in the same transaction using the [Beacon Proxy pattern](https://docs.openzeppelin.com/contracts/3.x/api/proxy#BeaconProxy). The `PoolFactory` contract enables anyone to deploy a pool. Once you create a pool using this contract, it is automatically added to the registry. This makes the process simple and anyone can easily participate in creating and managing pools in the system.

#### Gov contracts

Each DAO comprises four distinct governance contracts, which perform specific roles.

The `GovPool` contract operates as a shared space where users participate in the governance process. They may suggest ideas, vote on proposals, earn rewards for their involvement and observe decisions being put into action. Furthermore, governance pools offer a diverse set of useful features:

* Within the DAO pool, users have the ability to delegate voting assets to one another. It is also possible to achieve the status of an expert by possessing a unique `ERC721Expert` NFT, allowing direct delegation from the pool treasury.
* Users can gain extra votes through the ownership of a specific `ERC721Multiplier` NFT, which functions like a coupon and expires after a certain period.
* The `VotePower` contract allows users to transform their votes. In its basic variant, it adjusts the user's vote power based on factors such as whether the user is an expert and the number of delegated treasury assets. This empowers users to amplify their votes and consequently earn greater rewards.
* Users have the option to submit specific proposals, such as the `DistributionProposal`, which distributes sent rewards among the voters based on their number of votes, and the `TokenSaleProposal`, enabling users to create token sales with certain parameters.

The `GovUserKeeper` contract acts as a secure custodian safeguarding users' funds during the voting process. It ensures the secure storage of funds and their judicious use exclusively for voting purposes. The system supports four token standards available for users to deposit and vote with: `ERC20`, `ERC721`, `ERC721Enumerable` and `ERC721Power` which is a unique token standard wherein each NFT has its own power that changes dynamically based on collateralization and time.

The `GovValidator` contract is mandatory for proposals that require a two-stage voting. After achieving quorum in the first stage, such proposals are moved to the validators voting. The validators list is managed by the each DAO itself. Validators use a specific `GovValidatorsToken` that incorporates a snapshot logic.

The `GovSettings` contract stores configuration settings defining how proposals are handled within the governance pool.

# Usage of subgraphs

Sometimes, there is a need to acquire complex data from contracts, making it impractical to create view functions for all cases. This is where subgraphs, leveraging The Graph technology, come into play. DeXe has three subgraphs that listen to contract events and aggregate data from them. You can easily query the desired data by sending GraphQL-like requests to our subgraphs.

### The All Interactions subgraph

The All Interactions subgraph stores information about each user or validator interaction with protocol smart contracts.

#### Examples

We can get user interactions with proposals in the specific DAO pool by the user transaction (there can be many interactions in one transaction). To do this, we need to get the transaction by its hash and filter interactions by the DAO pool contract address.

```graphql
{
  transactions(where: {id: "<tx hash>"}) {
    daoPoolProposalInteraction(where: {pool: "<pool address>"}) {
      interactionType
      totalVote
    }
  }
}
```

Or we can get all vests (deposits/withdrawals) from all users.

```graphql
{
  daoPoolVests {
    amount
    nfts
    transaction {
      id
      user
    }
  }
}
```

## The DAO Pools subgraph

The DAO Pools subgraph stores information about each DAO Pool. In this subgraph, you can find detailed information about pools, proposals, voters, voter delegation history, etc.

#### Examples

We can retrieve all unexecuted proposals. To do this, we need to filter out records where the execution time is not equal to 0.

```graphql
{
  proposals(where: {executionTimestamp_not: 0}) {
    proposalId
    quorum
    isFor
    currentVotesFor
    currentVotesAgainst
    description
    voters {
      totalVotes
      totalVotedProposals
      totalProposalsCreated
      totalMicropoolRewardUSD
      totalLockedFundsUSD
      totalDelegatedUSD
      totalClaimedUSD
      delegatorsCount
      delegateesCount
      currentVotesReceived
      currentVotesDelegated
    }
  }
}
```

Or we can get the delegation history in the specific DAO pool with delegated assets by the delegator's address.

```graphql
{
  voterInPoolPairs(
    where: {delegator_: {voter: "<user address>"}}
  ) {
    delegatedAmount
    delegatedNfts
    delegatedUSD
    delegatedVotes
    delegatee {
      voter {
        id
      }
    }
  }
}
```

## The DAO Validators subgraph

The DAO Validators subgraph contains details of each validator in the DAO Pool, along with their proposals and other information.

#### Examples

We can retrieve all internal validator proposals that a certain validator has voted on by filtering proposals based on the `isInternal` flag.

```graphql
{
  proposals(
    where: {voters_: {validator_: {validatorAddress: "<validator address>"}}, isInternal: true}
  ) {
    isInternal
    proposalId
    quorum
    totalVoteAgainst
    totalVoteFor
    voters {
      totalVoteAgainst
      totalVoteFor
      validator {
        validatorAddress
        balance
      }
    }
  }
}
```

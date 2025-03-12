# Glossary

Glossary of technical terms you may encounter in this documentation.

| Term                            | Definition                                                                                                                                          |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| DAO pool, Governance pool, Pool | The `GovPool` contract, which is an entry point to interact with one of the DAOs deployed on the protocol.                                          |
| Proposal                        | The operation that can be executed on behalf of the `GovPool` contract if the voting is successful.                                                 |
| Quorum                          | The minimum number of votes required for a proposal to be considered executable.                                                                    |
| Actions for, Actions against    | Lists of operations to be executed if the proposal reaches the for or against quorum respectively.                                                  |
| Executor                        | The contract or EOA that will be called on behalf of the `GovPool` contract during proposal execution. Each action is associated with one executor. |
| Main executor                   | The executor of the last action in the list.                                                                                                        |
| Delegator                       | The user delegating assets to another user.                                                                                                         |
| Delegatee, Micropool            | The user receiving delegated tokens.                                                                                                                |
| Treasury                        | The balance of the `GovPool` contract.                                                                                                              |
| Personal balance                | The user's balance, which consists of assets deposited by themselves.                                                                               |
| Micropool balance               | The user's balance, which consists of assets delegated by other users.                                                                              |
| Treasury balance                | The user's balance, which consists of assets delegated from treasury.                                                                               |
| Metagovernance                  | The hierarchical structure where one DAO's vote influences decisions in another DAO.                                                                |
| Proposal settings               | Specific parameters set for each proposal by its main executor (`duration`, `quorum`, etc.).                                                        |
| NFT power                       | The quantity of votes given by certain NFT.                                                                                                         |
| Raw power                       | The total number of user votes before applying any transformation, calculated as the sum of the number of tokens and NFT powers user voted with.    |
| Voting power, Power             | The raw power after transformation directly involved in achieving quorum.                                                                           |
| Validators                      | Trusted addresses voting on the second stage of voting. `GovValidatorsToken` holders.                                                               |
| Tier                            | The token sale within the `TokenSaleProposal`.                                                                                                      |

# Transaction Handling in a Proof of Stake (PoS) Blockchain

## 1. Transaction Broadcast in PoS Network

- A user creates and signs a transaction using their private key.
- The signed transaction is broadcasted to the user’s connected peers in the P2P network.
- These peers validate the transaction’s syntax and signature, then propagate it further to their connected peers.
- This gossip-based propagation continues until most or all validators receive the transaction.
- Each validator maintains a **mempool** containing all pending transactions.

---

## 2. Validators and Block Proposal Selection

- Validators are participants who have staked tokens as collateral.
- The PoS consensus protocol selects a **block proposer (leader)** for each new block.
- Selection is often random but weighted by the amount of stake held by validators.
- The chosen proposer creates a new block with a selection of transactions from their mempool.

---

## 3. Transaction Sorting in Validator Mempools

Validators sort transactions to maximize block rewards and maintain transaction order:

- Transactions are sorted by **gas price (fee per gas unit)** in descending order to prioritize the most profitable ones.
- For transactions originating from the same account, they are sorted by **nonce in ascending order** to ensure correct execution order.
- The proposer selects transactions fitting within the block’s **gas limit** or size limit.

---

## 4. Block Proposal and Consensus Process

- The proposer packages the selected transactions into a block and broadcasts it to all validators.
- Validators independently validate the block’s transactions and state transitions.
- Validators participate in a multi-step voting process, typically including:
  - **Pre-vote**: Validators signal approval or rejection of the proposed block.
  - **Pre-commit**: Validators commit to the block if a supermajority supports it.
- Once a supermajority (e.g., > 2/3 of stake-weighted votes) is reached, the block is **finalized** and appended to the canonical blockchain.

---

## 5. Transaction Confirmation and State Update

- Transactions within a finalized block are considered **confirmed** and irreversible.
- The blockchain state (account balances, smart contract states) is updated accordingly.
- Finality in PoS is often **instant**, meaning finalized blocks cannot be reverted without severe penalties (e.g., slashing).

---

## 6. Layer 2 Interaction with PoS Layer 1

- Layer 2 solutions (e.g., zkRollups, Optimistic Rollups) batch multiple transactions off-chain.
- The Layer 2 operator submits proofs or transaction batch summaries to the Layer 1 mempool as special transactions.
- PoS validators treat these submissions like any other transaction:
  - Included in blocks based on gas price and proposer selection.
  - Validated and finalized through the PoS consensus mechanism.
- Layer 1 finality ensures Layer 2 state updates are secured on-chain.

---

## Summary Table

| Stage                | Description                                                |
|----------------------|------------------------------------------------------------|
| Transaction broadcast| User tx propagated via P2P gossip network                   |
| Validator selection  | Proposer chosen randomly, weighted by stake                 |
| Mempool sorting      | Sort txs by gas price desc, nonce ascending per account    |
| Tx selection         | Select highest paying, ordered txs fitting block gas limit  |
| Consensus voting     | Validators pre-vote, pre-commit; supermajority finalizes   |
| Finality             | Instant finality ensures block and txs cannot be reverted  |
| Layer 2 commit       | L2 batches submitted as txs to Layer 1, finalized by PoS   |

---

This process ensures efficient, secure transaction processing with fair validator participation and rapid finality on PoS blockchains.
```mermaid
flowchart TD
    %% User and Network
    U[User creates and signs transaction]
    U -->|Broadcast tx to peers| P2P[P2P Gossip Network]
    P2P -->|Propagate tx| V1[Validator Node 1]
    P2P -->|Propagate tx| V2[Validator Node 2]
    P2P -->|Propagate tx| V3[Validator Node 3]
    
    %% Validators receive and mempool
    subgraph Mempools
        direction TB
        V1 --> M1[Mempool 1: Store tx, validate signature and nonce]
        V2 --> M2[Mempool 2: Store tx, validate signature and nonce]
        V3 --> M3[Mempool 3: Store tx, validate signature and nonce]
    end

    %% Validator selection
    subgraph Validator Selection
        direction LR
        StakePool[Validators stake tokens]
        RandomSelection[Random selection weighted by stake]
        StakePool --> RandomSelection
        RandomSelection --> Proposer[Block Proposer Selected]
    end

    %% Proposer selects transactions
    Proposer -->|Access mempool| M1
    Proposer --> SortTx[Sort transactions by gas price descending and nonce ascending]
    SortTx --> SelectTx[Select transactions fitting block gas limit]

    %% Propose block
    SelectTx --> ProposedBlock[Create Proposed Block with transactions]
    ProposedBlock --> BroadcastBlock[Broadcast block to validators]

    %% Consensus Voting Steps
    subgraph Consensus Voting
        direction TB
        BroadcastBlock --> ValidateBlock[Validators validate block and transactions]
        ValidateBlock --> PreVote[Pre-vote: Approve or reject block]
        PreVote --> PreCommit[Pre-commit: Commit if majority supports]
        PreCommit --> Finalize[Block Finalized with supermajority]
    end

    %% Finality and State Update
    Finalize --> StateUpdate[Update blockchain state: balances and contracts]
    StateUpdate --> TxConfirmed[Transactions confirmed and irreversible]

    %% Layer 2 Interaction
    subgraph Layer 2 Commit to Layer 1
        direction LR
        L2Operator[Layer 2 operator batches many transactions off-chain]
        L2Operator --> SubmitBatch[Submit batch or zkProof as Layer 1 transaction]
        SubmitBatch --> M1
    end

```

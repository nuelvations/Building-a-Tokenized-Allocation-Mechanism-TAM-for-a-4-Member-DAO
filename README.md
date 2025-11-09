**— Building a Tokenized Allocation Mechanism (TAM) for a 4-Member DAO as a tutorial to explain the octantv2**

When you think of community decision-making in Web3, you’re often met with one core principle — **fairness through decentralization**.

Before Octant v2, **Octant V1** introduced the foundation of decentralized funding coordination — a transparent, non-custodial protocol that let communities allocate rewards collectively using Ethereum smart contracts. It established the core model for **onchain governance and public goods funding**, paving the way for the more modular and extensible **Octant V2**.

---

## Understanding Octant v2: Public Infrastructure for Sustainable Growth

**Octant v2** is open, public infrastructure for sustainable onchain growth.

It’s free for anyone to use — like public roads or libraries — but built for crypto funding. Its mission is to help projects and DAOs grow sustainably, by **recycling onchain yield** from deployed capital into public goods and community-driven initiatives.

### Core Components

- **Smart Contracts:** The backbone of Octant — enforcing allocation rules, yield routing, and distribution logic trustlessly.
- **Tooling:** Developer and user interfaces that make interacting with onchain funding easier.
- **Onchain Yield:** Returns generated from DeFi strategies, automatically redirected to support ecosystem projects.
- **Funding Vaults:** ERC-4626-compliant vaults that hold users’ tokens, generate yield, and manage donation streams.

Users can **select DeFi protocols** (like Aave or Yearn) for yield generation, **define a donation address** (direct or via payment splitters), and optionally **run allocation rounds** — community-based processes that let token holders decide how yield is distributed.

This creates a transparent pipeline: capital in vaults generates yield → yield is redirected → funding flows to DAOs, open-source builders, and ecosystem growth.

By separating **principal** from **yield**, Octant empowers long-term, sustainable impact without sacrificing users’ underlying capital.
![mermaid-diagram](https://github.com/user-attachments/assets/1886030b-2b12-47b4-a967-d860b9c506af)

The system's core building blocks are its vaults, which handle deployed tokens, strategies, and that connect with yield generation sources. Octant funding vaults support different kinds of assets, including base tokens (non-yield-bearing tokens), as well as yield-bearing tokens (both rebasing and non-rebasing ones). 

Using yield-bearing tokens can be thought of as token restaking for funding purposes, but without exposing the tokens to extra counterparty risks. After the tokens got deployed, a user receives back yield-donating tokens that represent the principal that is used in the funding vault. When a strategy generates profit, the system mints new shares that represent the yield and sends them to a designated donation address rather than to the user. 

The donation address can be a payment splitter that distributes these funds to a range of recipients. This distribution can be direct - sending funds straight to a project or routed through allocation mechanisms such as quadratic funding (QF), quadratic voting (QV), or other customizable funding rounds. 

A key difference between Octant and traditional ERC-4626 vaults lies in how yield is treated. In most vaults, yield accrues to the depositor's balance. Octant instead diverts the yield to funding purposes, effectively turning yield into a public good. This creates something akin to a "Kickstarter powered by yield", where users retain their principal and use the generated yield to back initiatives they care about. For example, a USDC funding vault may run multiple strategies to generate yield; whenever profit is realized, a network participant called a keeper triggers a report, and the newly created shares are directed to the donation address for further distribution to designated recipients.
By separating principal from yield, Octant v2 enables a sustainable, long-term funding model that empowers communities to support open-source development, public goods, and other impactful initiatives. Its design allows for flexible allocation methods and priotizes that the share-to-asset ratio generally remains one-to-one. This architecture creates a robust, transparent funding network that aligns incentives between capital providers and projects, making it possible to channel the power of yield toward meaningful collective impact

---## Introducing the Tokenized Allocation Mechanism (TAM)

The Octant Protocol takes this to another level with its **Tokenized Allocation Mechanism (TAM)**, 

The **Tokenized Allocation Mechanism (TAM)** is how Octant formalizes collective funding decisions. It allows DAOs and communities to **tokenize participation**, define clear allocation logic, and automate governance outcomes onchain.

Lets picture an instnaces guide to walks you through building a **4-member DAO TAM contract** — an ERC-20-based implementation that follows Octant’s architectural model. Each participant’s voting power is derived from token holdings, and governance decisions follow structured phases: **Registration, Proposal Creation, Voting, Finalization, Allocation, and Queuing**.

By the end, you’ll understand how TAM ties into the **Octant base contracts**, how to structure your own custom allocation mechanism, and how to extend it to larger DAOs or other tokenized collectives.

## Step 1: Registration

### Concept

Before any votes can be cast, each DAO member must **register**.

This registration step ensures that every address is recognized and bound to a single unit of voting eligibility, preventing Sybil attacks or multiple sign-ups from the same account.

Even though the DAO uses ERC-20 tokens for weighted voting, registration acts as an *eligibility gate* — confirming who can actually participate.

Octant v2 is open public infrastructure for sustainable growth.

### Code Snippet
mapping(address => bool) public registered;
address[] public registeredVoters;

function register() external {
    require(!registered[msg.sender], "Already registered");
    registered[msg.sender] = true;
    registeredVoters.push(msg.sender);
}
This function defines a `register` function that allows Ethereum addresses to register. 

Key aspects:

- **Restriction:** Each address can only register once.
- **Mechanism:** It uses a boolean mapping `registered` to track registration status.
- **Process:** If an address isn't already registered, it marks the address as registered, adds it to an array called `registeredVoters`, and emits a `Registered` event.
- **Error Handling:** It uses `require` to prevent multiple registrations from the same address, reverting the transaction if an address tries to register again.

---

## Step 2: Proposal Creation

### Concept

Any registered member can create a proposal.

Each proposal has:

- A title or identifier.
- A **deadline** (after which votes are invalid).
- Counters for `forVotes`, `againstVotes`, and `abstainVotes`.

This enforces time-bounded governance, ensuring DAO members act within defined decision windows.
### Code Snippet
function createProposal(string calldata description, uint256 deadline) external onlyRegistered {
    require(deadline > block.timestamp, "Invalid deadline");

    Proposal memory newProposal = Proposal({
        id: proposals.length,
        description: description,
        forVotes: 0,
        againstVotes: 0,
        abstainVotes: 0,
        deadline: deadline,
        finalized: false
    });

    proposals.push(newProposal);
    emit ProposalCreated(newProposal.id, description, deadline);
}

The `createProposal` function allows registered users to submit proposals with a voting deadline.

Here's the breakdown:

- **`function createProposal(string calldata description, uint256 deadline) external onlyRegistered`**: This declares a function named `createProposal` that takes a proposal `description` (a string) and a `deadline` (a Unix timestamp) as input. It's `external`, meaning it can be called from outside the contract, and `onlyRegistered`, meaning only registered users can call it.
- **`require(deadline > block.timestamp, "Invalid deadline")`**: This checks if the provided `deadline` is in the future. It must be later than the current block's timestamp. If not, it reverts the transaction with the message "Invalid deadline".
- **`Proposal memory newProposal = Proposal({...})`**: This creates a new `Proposal` struct in memory.
    - `id: proposals.length`: The proposal's ID is set to the current number of proposals.
    - `description: description`: The proposal's description is set to the input `description`.
    - `forVotes: 0`, `againstVotes: 0`, `abstainVotes: 0`: The initial vote counts are set to 0.
    - `deadline: deadline`: The proposal's deadline is set to the input `deadline`.
    - `finalized: false`: The proposal is initially not finalized.
- **`proposals.push(newProposal)`**: This adds the newly created proposal to the `proposals` array, making it an official proposal in the system.
- **`emit ProposalCreated(newProposal.id, description, deadline)`**: This emits an event named `ProposalCreated` containing the proposal's ID, description, and deadline. This allows external listeners to track new proposals being created.

- ## Step 3: Voting

### Concept

Each registered address can vote once per proposal.

Voting power is determined by **ERC-20 token balances**, typically using **snapshot balances** to capture fair voting power at the moment the proposal starts — preventing last-minute token transfers that could skew results.

Members can vote **For**, **Against**, or **Abstain**.

### Code Snippet
// Voting function using ERC-20 token-based power
function vote(uint256 proposalId, VoteType choice) external onlyRegistered {
    Proposal storage proposal = proposals[proposalId];
    require(block.timestamp < proposal.deadline, "Voting closed");
    require(!hasVoted[proposalId][msg.sender], "Already voted");

    uint256 votes = governanceToken.balanceOf(msg.sender);
    require(votes > 0, "No voting power");

    if (choice == VoteType.For) proposal.forVotes += votes;
    else if (choice == VoteType.Against) proposal.againstVotes += votes;
    else proposal.abstainVotes += votes;

    hasVoted[proposalId][msg.sender] = true;
    emit VoteCast(msg.sender, proposalId, choice, votes);
    Let’s explain the provided Solidity code for a voting function:

**The Purpose:**

This `vote` function allows registered users to cast their vote on a specific proposal, with their voting power determined by the balance of their ERC-20 governance tokens.

**Explanation:**

1. **`function vote(uint256 proposalId, VoteType choice) external onlyRegistered {`**:
    - This defines the `vote` function, which takes two arguments:
        - `proposalId`: The ID of the proposal the user wants to vote on.
        - `choice`: An enum (`VoteType`) representing the user's vote (For, Against, or Abstain).
    - `external`: Indicates the function is called from outside the contract.
    - `onlyRegistered`: A modifier (not shown in the snippet, but assumed) that restricts the function call to registered users only.
2. **`Proposal storage proposal = proposals[proposalId];`**:
    - Retrieves the proposal data from the `proposals` mapping (likely a `mapping(uint256 => Proposal)`), using the provided `proposalId`. `Proposal storage proposal` creates a storage pointer to avoid copying the entire struct.
3. **`require(block.timestamp < proposal.deadline, "Voting closed");`**:
    - Checks if the current block timestamp is before the proposal's deadline. If the deadline has passed, the voting is closed, and the transaction reverts with the error message "Voting closed".
    - 4. **`require(!hasVoted[proposalId][msg.sender], "Already voted");`**:
    - Checks if the user (`msg.sender`) has already voted on this proposal (`proposalId`). The `hasVoted` mapping (likely a `mapping(uint256 => mapping(address => bool))`) tracks who has voted on which proposal. If the user has already voted, the transaction reverts with the error message "Already voted".
5. **`uint256 votes = governanceToken.balanceOf(msg.sender);`**:
    - Retrieves the user's balance of the governance token using the `balanceOf` function of the `governanceToken` contract (likely an ERC-20 contract). This balance represents the user's voting power.
6. **`require(votes > 0, "No voting power");`**:
    - Ensures that the user has a positive balance of governance tokens. If the user has no tokens, the transaction reverts with the error message "No voting power".
7. **`if (choice == VoteType.For) proposal.forVotes += votes;`**:
    - If the user's choice is `VoteType.For`, the user's voting power (`votes`) is added to the `forVotes` count of the proposal.
8. **`else if (choice == VoteType.Against) proposal.againstVotes += votes;`**:
    - If the user's choice is `VoteType.Against`, the user's voting power (`votes`) is added to the `againstVotes` count of the proposal.
9. **`else proposal.abstainVotes += votes;`**:
    - If the user's choice is `VoteType.Abstain`, the user's voting power (`votes`) is added to the `abstainVotes` count of the proposal.
10. **`hasVoted[proposalId][msg.sender] = true;`**:
    - Records that the user (`msg.sender`) has voted on this proposal (`proposalId`) by setting the corresponding value in the `hasVoted` mapping to `true`.
11. **`emit VoteCast(msg.sender, proposalId, choice, votes);`**:
    - Emits a `VoteCast` event, which includes the voter's address (`msg.sender`), the proposal ID (`proposalId`), the vote choice (`choice`), and the voting power used (`votes`). Events are used to log and track activity on the blockchain.
## Step 4: Quorum & Finalization

### Concept

Proposals are finalized after their deadline passes.

To pass, a proposal must meet **two quorum conditions**:

1. `forVotes ≥ quorumFraction * registeredVoters`
2. `forVotes > againstVotes`

This ensures community consensus rather than raw token dominance.

### Code Snippet
// Finalization based on quorum rules
function finalizeProposal(uint256 proposalId) external {
    Proposal storage proposal = proposals[proposalId];
    require(block.timestamp >= proposal.deadline, "Deadline not reached");
    require(!proposal.finalized, "Already finalized");

    uint256 quorum = (quorumFraction * registeredVoters.length) / 100;
    if (proposal.forVotes >= quorum && proposal.forVotes > proposal.againstVotes) {
        proposal.finalized = true;
        passingProposals.push(proposalId);
        emit ProposalPassed(proposalId);
    } else {
        proposal.finalized = true;
        emit ProposalFailed(proposalId);
    }
}
    
The `finalizeProposal` function determines the outcome of a proposal based on whether it met the quorum and majority requirements.

**Functionality:**

1. **Fetch Proposal:** Retrieves the proposal data using the `proposalId`.
2. **Check Deadline:** Ensures the current block timestamp is past the proposal's deadline. Reverts if the deadline hasn't been reached.
3. **Check Finalization Status:** Verifies the proposal hasn't already been finalized. Reverts if it has.
4. **Calculate Quorum:** Calculates the minimum number of votes required to reach quorum, based on `quorumFraction` and the total number of registered voters.
5. **Determine Outcome:**
    - If the proposal's `forVotes` are greater than or equal to the quorum *and* the `forVotes` are greater than the `againstVotes`, the proposal is considered passed.
        - The `finalized` flag is set to `true`.
        - The `proposalId` is added to the `passingProposals` array.
        - A `ProposalPassed` event is emitted.
    - Otherwise, the proposal is considered failed.
        - The `finalized` flag is set to `true`.
        - A `ProposalFailed` event is emitted.

In essence The function checks if enough people voted for the proposal and if more people voted for it than against it. If both conditions are met, the proposal passes; otherwise, it fails. The `finalized` flag prevents the proposal from being processed multiple times.
## Step 5: Allocation

### Concept

Once proposals are finalized, those that pass receive **token allocations** proportional to their total “forVotes” — relative to a fixed **budget**.

This is the heart of the **Tokenized Allocation Mechanism** (TAM):

rewarding proposals that capture more community support with more resources, all transparently and onchain.

### Code Snippet
// Allocate budget proportionally among passing proposals
function allocate(uint256 totalBudget) external {
    uint256 totalForVotes = 0;

    for (uint256 i = 0; i < passingProposals.length; i++) {
        totalForVotes += proposals[passingProposals[i]].forVotes;
    }

    for (uint256 i = 0; i < passingProposals.length; i++) {
        Proposal storage p = proposals[passingProposals[i]];
        uint256 share = (p.forVotes * totalBudget) / totalForVotes;
        allocations[p.id] = share;
        emit Allocated(p.id, share);
    }
}

This Solidity function `allocate` distributes a `totalBudget` among a set of `passingProposals` proportionally to their respective `forVotes`.

First, it calculates `totalForVotes` by summing the `forVotes` of all proposals in the `passingProposals` array.

Then, it iterates through the `passingProposals` array again. For each proposal, it calculates its `share` of the `totalBudget` by multiplying the proposal's `forVotes` by the `totalBudget` and dividing by `totalForVotes`.  This share is then assigned to the `allocations` mapping, using the proposal's `id` as the key. Finally, an `Allocated` event is emitted, indicating the proposal's `id` and its allocated `share`.

## Step 6: Permissionless Queuing

### Concept

Octant’s TAM emphasizes *composability* — anyone can queue allocations or execute results, reinforcing decentralization.

This permissionless design ensures no central authority controls the outcome execution.

### Code Snippet
// Permissionless queuing of finalized allocations
function queue(uint256 proposalId) external {
    require(proposals[proposalId].finalized, "Not finalized");
    require(allocations[proposalId] > 0, "No allocation");

    emit Queued(proposalId, allocations[proposalId]);
}

The Solidity function `queue(uint256 proposalId)` is used to enqueue a finalized allocation for a given proposal;

- **`function queue(uint256 proposalId) external { ... }`**: This declares an external function named `queue` that takes a `uint256` (unsigned integer) named `proposalId` as input. `external` means this function can only be called from outside the contract.
- **`require(proposals[proposalId].finalized, "Not finalized");`**: This line checks if the proposal with the given `proposalId` has been finalized. It accesses a mapping or array called `proposals` (presumably defined elsewhere in the contract), retrieves the entry for the given `proposalId`, and checks its `finalized` property (likely a boolean). If `proposals[proposalId].finalized` is false, the `require` statement fails, and the transaction reverts with the message "Not finalized". This ensures that only finalized proposals can be queued.
- **`require(allocations[proposalId] > 0, "No allocation");`**: This line checks if there is a non-zero allocation for the given `proposalId`. It accesses a mapping or array called `allocations` (presumably defined elsewhere in the contract) and retrieves the allocation amount for the given `proposalId`. If `allocations[proposalId]` is not greater than 0, the `require` statement fails, and the transaction reverts with the message "No allocation". This ensures that only proposals with a positive allocation can be queued.
- **`emit Queued(proposalId, allocations[proposalId]);`**: If both `require` statements pass, this line emits an event called `Queued`. The event includes the `proposalId` and the corresponding `allocations[proposalId]` as arguments. Events are used to log and communicate state changes on the blockchain. In this case, it signals that the proposal has been successfully queued, along with its allocated amount.
- ## Full Contract: `DAOTAM4.sol`

Below is the full, self-contained ERC-20-based Tokenized Allocation Mechanism for a four-member DAO, built on top Octant’s architecture.

- **Full Contract** = the *entire Solidity file* that can be compiled and deployed onchain (e.g., using Remix, Hardhat, or Foundry).
- **`DAOTAM4.sol`** = the *filename* given to that complete implementation.
    
    **DAO** — because it’s designed for decentralized governance.
    
    **TAM** — because it implements Octant’s Tokenized Allocation Mechanism.
    
    **“4”** — because this particular version is designed for a **four-member DAO** (though it can scale).
    
    **“.sol”** — the standard Solidity source file extension.
  
  .../ SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract DAOTAM4 {
    IERC20 public governanceToken;
    uint256 public quorumFraction;
    address[] public registeredVoters;

    enum VoteType { For, Against, Abstain }

    struct Proposal {
        uint256 id;
        string description;
        uint256 forVotes;
        uint256 againstVotes;
        uint256 abstainVotes;
        uint256 deadline;
        bool finalized;
    }

    Proposal[] public proposals;
    mapping(address => bool) public registered;
    mapping(uint256 => mapping(address => bool)) public hasVoted;
    mapping(uint256 => uint256) public allocations;
    uint256[] public passingProposals;

    event Registered(address voter);
    event ProposalCreated(uint256 id, string description, uint256 deadline);
    event VoteCast(address voter, uint256 proposalId, VoteType choice, uint256 votes);
    event ProposalPassed(uint256 proposalId);
    event ProposalFailed(uint256 proposalId);
    event Allocated(uint256 proposalId, uint256 share);
    event Queued(uint256 proposalId, uint256 share);

    constructor(address _token, uint256 _quorumFraction) {
        governanceToken = IERC20(_token);
        quorumFraction = _quorumFraction;
    }

    modifier onlyRegistered() {
        require(registered[msg.sender], "Not registered");
        _;
    }

    function register() external {
        require(!registered[msg.sender], "Already registered");
        registered[msg.sender] = true;
        registeredVoters.push(msg.sender);
        emit Registered(msg.sender);
    }

    function createProposal(string calldata description, uint256 deadline) external onlyRegistered {
        require(deadline > block.timestamp, "Invalid deadline");
        Proposal memory newProposal = Proposal({
            id: proposals.length,
            description: description,
            forVotes: 0,
            againstVotes: 0,
            abstainVotes: 0,
            deadline: deadline,
            finalized: false
        });
        proposals.push(newProposal);
        emit ProposalCreated(newProposal.id, description, deadline);
    }

    function vote(uint256 proposalId, VoteType choice) external onlyRegistered {
        Proposal storage proposal = proposals[proposalId];
        require(block.timestamp < proposal.deadline, "Voting closed");
        require(!hasVoted[proposalId][msg.sender], "Already voted");
        uint256 votes = governanceToken.balanceOf(msg.sender);
        require(votes > 0, "No voting power");

        if (choice == VoteType.For) proposal.forVotes += votes;
        else if (choice == VoteType.Against) proposal.againstVotes += votes;
        else proposal.abstainVotes += votes;

        hasVoted[proposalId][msg.sender] = true;
        emit VoteCast(msg.sender, proposalId, choice, votes);
    }

    function finalizeProposal(uint256 proposalId) external {
        Proposal storage proposal = proposals[proposalId];
        require(block.timestamp >= proposal.deadline, "Deadline not reached");
        require(!proposal.finalized, "Already finalized");

        uint256 quorum = (quorumFraction * registeredVoters.length) / 100;
        if (proposal.forVotes >= quorum && proposal.forVotes > proposal.againstVotes) {
            proposal.finalized = true;
            passingProposals.push(proposalId);
            emit ProposalPassed(proposalId);
        } else {
            proposal.finalized = true;
            emit ProposalFailed(proposalId);
        }
    }

    function allocate(uint256 totalBudget) external {
        uint256 totalForVotes = 0;
        for (uint256 i = 0; i < passingProposals.length; i++) {
            totalForVotes += proposals[passingProposals[i]].forVotes;
        }

        for (uint256 i = 0; i < passingProposals.length; i++) {
            Proposal storage p = proposals[passingProposals[i]];
            uint256 share = (p.forVotes * totalBudget) / totalForVotes;
            allocations[p.id] = share;
            emit Allocated(p.id, share);
        }
    }
    
function queue(uint256 proposalId) external {
        require(proposals[proposalId].finalized, "Not finalized");
        require(allocations[proposalId] > 0, "No allocation");
        emit Queued(proposalId, allocations[proposalId]);
    }
}


This line of codes  signals the **complete Solidity implementation** of the Tokenized Allocation Mechanism (TAM) — not just snippets or modular fragments.

It’s the **final, deployable smart contract** that combines all the earlier steps (Registration, Proposal Creation, Voting, Quorum/Finalization, Allocation, and Queuing) into one coherent piece of code.

## Integration with Octant Base Contracts

---

To integrate this TAM with the broader **Octant protocol**, extend:

- `BaseFundingMechanism.sol` — implement hooks like `_processVote()` and `_allocate()`.
- Register your TAM in Octant’s registry.
- Optionally link it to `FundingRoundFactory.sol` for automatic funding round creation.

This ensures your DAO’s allocation logic is Octant-compliant, upgradable, and composable with the ecosystem’s yield vaults.
## Conclusion

Through the `DAOTAM4.sol` example, we’ve illustrated how Octant’s **Tokenized Allocation Mechanism** can be implemented at a DAO level — from registration to allocation — using ERC-20 voting power and quorum logic.

But TAM is only one dimension of what **Octant v2** enables. Developers can use its open infrastructure to:

- Create **custom funding mechanisms** beyond voting (e.g., quadratic funding, retroactive rewards).
- Design **yield-backed ecosystems**, where token yield autonomously funds builders and public goods.
- Launch **composable DAOs**, integrating onchain governance with real yield streams.

Octant v2 is more than a protocol — it’s an open coordination layer for the onchain economy, empowering communities to direct financial flows toward what truly matters: **sustainable, transparent, decentralized growth**.
        


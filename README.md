# ARCANA - Architecture of the QA Wallet State Deck ♠️


The Ledger passphrase feature can be used to derive **multiple independent wallet spaces from the same seed**.

Each passphrase effectively creates a **separate wallet space**:

```text
Seed S + Passphrase A → Puppet State Book (QA shared)
Seed S + Passphrase B → State / Activity Generator
Seed S + Passphrase C → Master Treasury
```

Even though all wallets originate from the same seed, each passphrase generates **a completely different set of accounts and addresses**.

```text
S + A ≠ S + B ≠ S + C
```

To make these wallet states usable and shareable across QA and Engineering, seeds can be stored on **Ledger Recovery Key (LRK) cards**, which act as physical carriers of reproducible wallet states.

This makes complex wallet states **durable, shareable, and reusable across QA workflows**, instead of being ephemeral and tied to individual environments / scopes.

A collection of such cards naturally forms a **state card deck ♠️**, where each card represents a distinct wallet state.

This allows each card of the QA state deck to separate **testing states, state generation, and treasury management**, while maintaining a single root seed.

---

## System Model (Roles)

### Puppet States (Passphrase A)

**Purpose** : Provide **QA-shared testing states**.

**Characteristics**

- Seeds may be shared with QA and Software engineers (for debugging purposes)
    
- Wallet states intentionally crafted for testing
    
- Low balances
    
- Used to reproduce **state-dependent bugs**
    

**Example uses**

- fragmented balances
    
- large transaction histories
    
- dust-heavy wallets
    
- unusual account structures
    

These wallets represent the **public testing surface of the card deck ♠️**.

---

### State Generator (Passphrase B)

**Purpose** : Generate **transaction activity and interaction patterns** used to build and evolve puppet states. 
  
**Characteristics**  
  
- Seed **not shared with QA or engineers**  
- Passphrase never shared  
- Used to trigger interactions that mutate wallet states  
- Acts as an **activity source**, not a state itself  
- Interacts with both puppet wallets and external systems (DEX, staking, etc.)  
  
**Example uses**  
  
- sending transactions to puppet wallets  
- creating long interaction histories  
- fragmenting balances (dust generation)  
- interacting with smart contracts (swaps, approvals, staking, bridging)  
- simulating realistic user behavior  
  
**Example activity flow**
  
```text  
State Generator (B)  
↓ interacts  
Puppet Wallet (A)
```

This allows puppet wallets to gradually accumulate **realistic transaction histories and complex structures** through interactions.

---

### Master Treasury (Passphrase C)

**Purpose** : Act as the **primary funding source** for the QA card deck ♠️.

**Characteristics**

- Passphrase never shared
    
- Holds the majority of funds
    
- Used only to fund generator or puppet wallets
    

**Example flow**

```text
Master Treasury (C)
↓ funds
State Generator (B)
↓ interacts
Puppet Wallets (A)
```

This ensures that the treasury remains isolated from QA-shared wallets.

---

## Failure Model (Recovery + Leak)

If a puppet state card is compromised:  
  
- Only the **public puppet seed** is exposed  
- Funds remain limited  
- The **state generator wallet** can regenerate activity  
- The **master treasury** can refill balances  
  
Because activity can occur **between passphrase-derived wallets**, transaction histories can be rebuilt without starting completely from scratch (see `Passphrase Rotation Strategy`).  
  
Example interaction graph:  
  
```text  
A ↔ B  
A ↔ C  
B ↔ C
```

This interaction model ensures that **state can be reconstructed through controlled flows**, rather than lost entirely.

---
## Value (QA Impact)

Using multiple passphrases allows **each card of the QA state deck to behave like a living system, where state emerges from continuous interactions between its parts**, rather than a static snapshot.  
  
>Because wallet state evolves both through intentional interactions and everyday QA activity, this approach allows that complexity to be **generated and accumulated on shared seeds**, rather than lost across fragmented testing wallets.

```
Generator → creates interactions
QA → adds organic noise
System → accumulates both
```
  
Benefits include:  
  
- separation of **treasury, activity generation, and test states**  
- safer sharing of puppet states with QA or developers  
- ability to **rebuild complex states** after a compromise  
- creation of **organic transaction histories**  
- testing of realistic multi-wallet interaction patterns  
- improved coverage of:  
	- synchronization (Add Accounts)  
	- transaction flows  
	- state reconstruction logic (Ledger Sync)  
  
In practice, each state card ♠️ of the card deck becomes a **self-contained state ecosystem**, where wallet states evolve, interact, and accumulate complexity over time.

---
## Evolution Model (Rotation)

The multi-passphrase architecture also enables a **role rotation strategy** in case a seed becomes compromised.


Instead of rebuilding complex wallet states from scratch, roles can simply **shift upward within the passphrase hierarchy**.

Initial structure:

```
Seed S + Passphrase C → Master Treasury
Seed S + Passphrase B → State Generator
Seed S + Passphrase A → Puppet State (QA shared)
```

If a puppet seed is compromised, the system rotates:

```
Generator (B) → becomes new Puppet
Master (C) → becomes new Generator
New Passphrase (D) → becomes new Master Treasury
```

Resulting structure:

```
Seed S + Passphrase D → Master Treasury
Seed S + Passphrase C → State Generator
Seed S + Passphrase B → Puppet State
```

Key idea:

- **Only the Master Treasury requires minimal state complexity**
    
- **Generator and Puppet wallets already contain transaction history and structure**
    

Because of this, rotation allows the QA card deck to **preserve complex wallet states** while restoring security.

---

### Anticipating the First Public Seed Leak

In practice, the **first puppet card will likely be backed by a seed without passphrase**, since it will initially be shared with QA in order to bootstrap the deck.

A leak of that seed is therefore **plausible**.

Two factors increase this risk:

- some developers who may need to debug state-dependent issues are **fully remote**
    
- some QA engineers are also **remote**, and do not have immediate access to the **physical LRK card deck stored at the 106 office**
    

In those situations, the operational temptation may arise to **share additional passphrases** instead of rebuilding complex wallet states from scratch.

This would introduce a dangerous pattern:

Seed shared → Passphrase shared

which effectively collapses the separation between wallet domains.

The rotation strategy explicitly **avoids this trap**.

Instead of exposing additional secrets, the system simply **rotates roles upward**, keeping passphrases private while preserving the accumulated state complexity.

This ensures that:

- the **seed can remain stable**
    
- passphrases remain **strictly scoped by role**
    
- the team never needs to **expose additional secrets** in order to preserve valuable wallet states

---
## Operational Benefits

This rotation model provides several advantages:

- No need to **rebuild transaction history from scratch**
    
- Complex wallet states are **preserved across rotations**
    
- Security incidents trigger a **controlled role shift**, not a full reset
    
- QA testing can **continue with minimal disruption**
    

After rotation:

- the new puppet wallet can continue being used for QA testing
    
- the new generator wallet can generate additional activity
    
- the new master treasury remains isolated
    

Over time, this system allows the QA state card deck to evolve while maintaining **operational and financial resilience against seed compromise**.

---

## Concrete Instantiation (Arcana deck)

```
Arcana State Deck ♠️
│
├── The Fool
|   State profile: fresh / minimal wallet state
│   ├── Fool Puppet (QA shared testing state)
│   ├── Fool Generator (activity / state generator)
│   └── Fool Master (treasury / funding source)
│
├── The Hermit
|   State profile: old wallet with long transaction history
│   ├── Hermit Puppet
│   ├── Hermit Generator
│   └── Hermit Master
│
├── The Tower
|   State profile: catastrophic or pathological wallet state
│   ├── Tower Puppet
│   ├── Tower Generator
│   └── Tower Master
│
└── ...
```

---

## State Completeness — Account Discovery vs Declarative State

### Problem

Ledger wallets do not expose a **complete view of their state by default**.

Account visibility depends on **discovery mechanisms** (e.g. “Add Account”), which are:

- user-driven
    
- client-dependent (Ledger Live behavior)
    
- non-exhaustive
    
- not persisted outside the application state
    

This creates a structural gap:

Actual wallet state ≠ Observable wallet state

As a result:

- some accounts may exist but remain undiscovered
    
- state reconstruction after reset becomes **partial or inconsistent**
    
- QA cannot guarantee that a state card is **complete**
    

---

### Implication for the QA State Deck

Without additional structure, a state card ♠️:

- may silently miss accounts
    
- may not be reproducible after reset
    
- cannot be reliably shared across environments
    

This directly impacts:

- bug reproducibility
    
- state-dependent testing
    
- long-term state preservation
    

---

### Solution — Account Manifest (app.json)

To address this limitation, each state card can be extended with an **account manifest**:

State Card = Seed + Passphrase + Account Manifest

Where:

- **Seed + Passphrase** → defines the cryptographic space
    
- **Account Manifest (app.json)** → defines the expected account structure
    

The manifest explicitly declares:

- which accounts exist
    
- which currencies / derivation paths are used
    
- which accounts must be instantiated on restore
    

---

### Effect

This transforms the model from:

state = discovered (implicit, incomplete)

to:

state = declared (explicit, reproducible)

Benefits:

- deterministic reconstruction of wallet states
    
- no reliance on memory or manual discovery
    
- elimination of “hidden” or forgotten accounts
    
- improved portability across QA / Engineering environments
    

---

### New Responsibility

This introduces a consistency requirement:

Declared state (manifest) must match actual state

Which implies:

- maintaining the manifest when accounts evolve
    
- or generating it programmatically
    

Otherwise:

state drift = manifest ≠ real wallet state

---

### Conceptual Impact on Arcana

This addition introduces a new dimension to the model:

state completeness

A state card is no longer defined only by:

- its complexity
    
- its history
    
- its interactions
    

but also by:

- its **level of completeness / explicitness**
    

---

### Key Insight

A wallet state is not inherently fully observable.

By introducing an account manifest, Arcana shifts from:

capturing state

to:

defining state

This is what enables state cards to become:

- **durable**
    
- **shareable**
    
- **fully reproducible**
    

across the QA state deck ♠️.

---

Related :

[[Seeds Garden - Evolving Wallet States and Historical Traceability]]
[[Puppets Architecture - LRK State Library]]
[[Test Data Stewardship - State Library]]
[[State Complexity – Exploration Notes]]
[[Breadth vs Depth vs State Complexity Coverage]]
[[The Fool → (time)←The World - Generative and Archived Ecosystem States]]

---
CIP:
Title: Universal Query Layer
Category: Tools
Status: Proposed
Authors:
    - Vladimir Kalnitsky <klntsky@gmail.com>
Implementors: []
Discussions:
    - https://discord.gg/MU8vHAgmGy
Created: 2024-05-14
License: CC-BY-4.0
---

## Abstract
<!-- A short (\~200 word) description of the proposed solution and the technical issue being addressed. -->

A transport-agnostic query layer specification for use in dApps and wallets.

## Motivation: why is this CIP necessary?

See [CPS-12](https://github.com/cardano-foundation/CIPs/pull/625) for motivation.

## Specification

<!-- The technical specification should describe the proposed improvement in sufficient technical detail. In particular, it should provide enough information that an implementation can be performed solely on the basis of the design in the CIP. This is necessary to facilitate multiple, interoperable implementations. This must include how the CIP should be versioned, if not covered under an optional Versioning main heading. If a proposal defines structure of on-chain data it must include a CDDL schema in its specification.-->

### Existing Query Layer designs

There are two approaches to Cardano dApp development:

1. **Using customized chain followers**. A chain follower is a program that interacts with cardano-node and processes all incoming transactions, as well as rollbacks, to maintain consistent dApp-specific state. Example: [Carp](https://dcspark.github.io/carp/docs/intro/).

2. **Using general-purpose query layers**. General-purpose query layers allow to query blockchain data using a wide set of APIs that are not built with a particular dApp domain in mind. dApp state has to be constructed based on data returned from the queries. Examples: Blockfrost, Maestro.

The first approach allows for lower runtime resource consumption, but a general-purpose query layer has an advantage of being more easily reusable between dApps.

In this proposal, we are focusing on general-purpose querying only.

#### Handling of rollbacks

Transaction rollbacks are essential to blockchains: local node's view of the chain may be different from other nodes'. During conflict resolution, the node may issue a rollback event, that should be handled by dApps.

Customized chain followers, at least in principle, allow for "live" rollback handling: that is, a user-facing dApp can subscribe to a local view of a part of the UTxO set.

General purpose query layers can also handle rollbacks just fine, but they don't propagate rollback events to dApps, because they do not possess any dApp-specific info to determine if a dApp *needs* to handle a particular rollback. dApps that work with general-purpose query layers follow pull-based architecture, rather than event subscription-based, which means they just request data as needed, instead of reacting to blockchain events.

#### Error handling

Errors should be divided in two categories:

- domain errors
- transport errors (404, 500, etc)

This document should only cover domain errors.

### Methods

This section contains transport-agnostic method descriptions & their parameter lists.

The scope of this section is loosely based on a [comparison table for existing Cardano query layers](./Query_Layer_API_Comparison.md).
The goal is to make it so that the API could be implemented via simple adapters that transform requests and responses to the appropriate formats.

The payload formats used below are references to [CIP-0116 - Standard JSON encoding for Domain Types](https://cips.cardano.org/cip/CIP-0116), which specifies cardano domain types via a JSON schema.

#### UTxOs

##### By address

Request:

- address: `Address`
- slot?: `UInt64`

Response:

- `TransactionUnspentOutput[]`

##### By payment credential

Request:

- credential: `Credential`
- slot?: `UInt64`

Response:

- `TransactionUnspentOutput[]`

##### By staking credential

Request:

- credential: `Credential`
- slot?: `UInt64`

Response:

- `TransactionUnspentOutput[]`

##### By asset

Request:

- script_hash: `ScriptHash`
- asset_name: `AssetName`
- slot?: `UInt64`

Response:

- `TransactionUnspentOutput[]`

##### By transaction ID

- transaction_hash: `TransactionHash`
- slot?: `UInt64`

Response:

- `TransactionUnspentOutput[]`

#### Transaction

TODO: decide if we want mempool transactions, want to forbid these, or it doesn't matter.

##### By hash

Request:

- transaction_hash: `TransactionHash`

Response:

- `Transaction`

#### Script

##### Native Script by hash

Request:

- `ScriptHash`

Response:

- `NativeScript`

##### Plutus Script by hash

Request:

- `ScriptHash`

Response:

- `PlutusScript`

#### PlutusData

##### By hash

Request:

- hash: `DataHash`

Response:

- `PlutusData`

#### Reward history

##### By stake address

TODO

#### Block

##### By hash

Request:

- `BlockHash`

Response:

- `Block`

#### Epoch

TODO

#### Genesis

TODO

#### Metadata

##### By transaction hash

TODO

#### Protocol parameters

##### By epoch

Request:

- epoch: `UInt32`

Response:

- TODO: we should add `ProtocolParameters` to CIP-116

#### Stake pools

##### List of active stake pools

Request:

- slot?: `UInt64`

Response:

```
{
  pool_id: PoolPubKeyHash,
  live_stake: UInt64,
  active_stake: UInt64
}[]
```

##### Get pool by ID

Request:

- pool_pubkey_hash: `PoolPubKeyHash`

Response:

- `PoolParams`

### Transports

TODO: specify how to go from the description above to consumable API specs.

#### HTTP

TBD

#### WebSocket

TBD

#### Injected JavaScript object

TBD

## Rationale: how does this CIP achieve its goals?

<!-- The rationale fleshes out the specification by describing what motivated the design and what led to particular design decisions. It should describe alternate designs considered and related work. The rationale should provide evidence of consensus within the community and discuss significant objections or concerns raised during the discussion.

It must also explain how the proposal affects the backward compatibility of existing solutions when applicable. If the proposal responds to a CPS, the 'Rationale' section should explain how it addresses the CPS, and answer any questions that the CPS poses for potential solutions.
-->

## Path to Active

### Acceptance Criteria
<!-- Describes what are the acceptance criteria whereby a proposal becomes 'Active' -->

- [ ] There is at least one protocol adapter for any of the existing query layers that implements this spec, that can be run.
- [ ] There is at least one offchain library that implements a provider interface for this CIP, effectively making it usable with the protocol adapter in production.

### Implementation Plan
<!-- A plan to meet those criteria or `N/A` if an implementation plan is not applicable. -->

- [ ] Build at least one protocol adapter for any of the existing query layers that implements this spec
- [ ] Build at least one offchain library integration

<!-- OPTIONAL SECTIONS: see CIP-0001 > Document > Structure table -->

## Copyright
<!-- The CIP must be explicitly licensed under acceptable copyright terms.  Uncomment the one you wish to use (delete the other one) and ensure it matches the License field in the header: -->

This CIP is licensed under [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/legalcode).
<!-- This CIP is licensed under [Apache-2.0](http://www.apache.org/licenses/LICENSE-2.0). -->
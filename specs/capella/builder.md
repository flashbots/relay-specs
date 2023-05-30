# Relay specs

## Table of contents
<!-- TOC -->
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Introduction](#introduction)
- [Notation](#notation)
- [Custom Types](#custom-types)
- [Constants](#constants)
- [Containers](#containers)


<!-- END doctoc generated TOC please keep comment here to allow auto update -->
<!-- /TOC -->

## Introduction

This document presents the `mev-boost` relay specification. 

[`mev-boost`](https://boost.flashbots.net/) is an out-of-protocol mechanism to
connect Ethereum Proof-of-Stake validators to the external block-building market.
The [validator-side](https://github.com/flashbots/mev-boost) API is specified 
in the [builder specs](https://github.com/ethereum/builder-specs). The [relay-side](https://github.com/flashbots/mev-boost-relay) API is specified here in the relay specs. 

## Notation

Code snippets appearing in `this style` are to be interpreted as Python 3 code.

## Custom types

We define the following Python custom types for type hinting and readability:

| Name | SSZ equivalent | Description |
| - | - | - |
| `Slot` | `uint64` | a slot number |
| `ValidatorIndex` | `uint64` | a validator registry index |
| `Gwei` | `uint64` | an amount in Gwei |
| `Hash32` | `Bytes32` | a 256-bit hash |
| `BLSPubkey` | `Bytes48` | a BLS12-381 public key |
| `BLSSignature` | `Bytes96` | a BLS12-381 signature |
| `ExecutionAddress` | `Bytes20` | Address of account on the execution layer |
| `Transaction` | `ByteList[MAX_BYTES_PER_TRANSACTION]` | either a [typed transaction envelope](https://eips.ethereum.org/EIPS/eip-2718#opaque-byte-array-rather-than-an-rlp-array) or a legacy transaction|
| `WithdrawalIndex` | `uint64` | an index of a `Withdrawal` |
<!-- |`Gwei`|  -->

## Constants

The following values are (non-configurable) constants used throughout the specification.


| Name | Value |
| - | - |
| `BYTES_PER_LOGS_BLOOM` | `uint64(2**8)` (= 256) |
| `MAX_EXTRA_DATA_BYTES` | `2**5` (= 32) |
| `MAX_BYTES_PER_TRANSACTION` | `uint64(2**30)` (= 1,073,741,824) |
| `MAX_TRANSACTIONS_PER_PAYLOAD` | `uint64(2**20)` (= 1,048,576) |
| `MAX_WITHDRAWALS_PER_PAYLOAD` | `uint64(2**4)` (= 16) |


## Containers

*Note*: Fields missing in container instantiations default to their zero value.

### GET `​/relay​/v1​/builder​/validators` response type 

#### `ProposerDutiesResponse`

```python
class ProposerDutiesResponse(Container):
    data: List[BuilderGetValidatorsResponseEntry]
```

#### `BuilderGetValidatorsResponseEntry`

```python
class BuilderGetValidatorsResponseEntry(Container):
    slot: Slot
    validator_index: ValidatorIndex
    entry: SignedValidatorRegistration
```

#### `SignedValidatorRegistration`

```python
class SignedValidatorRegistration(Container):
    message: RegisterValidatorRequestMessage
    signature: BLSSignature
```



Slot           uint64                                  `json:"slot,string"`
	ValidatorIndex uint64                                  `json:"validator_index,string"`
	Entry          *boostTypes.SignedValidatorRegistration `json:"entry"`

### POST `​/relay​/v1​/builder​/blocks` request type 

#### `BuilderSubmitNewBlockRequest`

```python
class BuilderSubmitNewBlockRequest(Container):
    capella: SubmitBlockRequest
```

#### `SubmitBlockRequest`

```python
class SubmitBlockRequest(Container):
    message: BidTrace
    execution_payload: ExecutionPayload 
    signature: BLSSignature
```

#### `BidTrace`

```python
class BidTrace(Container):
    slot: Slot
    parent_hash: Hash32
    block_hash: Hash32
    builder_pubkey: BLSPubkey
    proposer_pubkey: BLSPubkey
    proposer_fee_recipient: ExecutionAddress
    gas_limit: uint64
    gas_used: uint64
    value: uint256
```

#### `ExecutionPayload` 

From [capella spec](https://github.com/ethereum/consensus-specs/blob/f7352d18cfb91c58b1addb4ea509aedd6e32165c/specs/capella/beacon-chain.md#executionpayload).

```python
class ExecutionPayload(Container):
    parent_hash: Hash32
    fee_recipient: ExecutionAddress
    state_root: Bytes32
    receipts_root: Bytes32
    logs_bloom: ByteVector[BYTES_PER_LOGS_BLOOM]
    prev_randao: Bytes32
    block_number: uint64
    gas_limit: uint64
    gas_used: uint64
    timestamp: uint64
    extra_data: ByteList[MAX_EXTRA_DATA_BYTES]
    base_fee_per_gas: uint256
    block_hash: Hash32 
    transactions: List[Transaction, MAX_TRANSACTIONS_PER_PAYLOAD]
    withdrawals: List[Withdrawal, MAX_WITHDRAWALS_PER_PAYLOAD] 
```

#### `Withdrawal`

From [capella spec](https://github.com/ethereum/consensus-specs/blob/f7352d18cfb91c58b1addb4ea509aedd6e32165c/specs/capella/beacon-chain.md#withdrawal).

```python
class Withdrawal(Container):
    index: WithdrawalIndex
    validator_index: ValidatorIndex
    address: ExecutionAddress
    amount: Gwei
```
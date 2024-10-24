# Deneb -- Relay Specs

## Table of contents

<!-- TOC -->
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Introduction](#introduction)
- [Custom Types](#custom-types)
- [Constants](#constants)
- [Containers](#containers)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
<!-- /TOC -->

## Introduction

This document presents the `mev-boost` relay specification for Deneb.

[`mev-boost`](https://boost.flashbots.net/) is an out-of-protocol mechanism to
connect Ethereum Proof-of-Stake validators to the external block-building
market. The [validator-side](https://github.com/flashbots/mev-boost) API is
specified in the [builder specs](https://github.com/ethereum/builder-specs). The
[relay-side](https://github.com/flashbots/mev-boost-relay) API is specified here
in the relay specs.

## Custom types

We define the following Python custom types for type hinting and readability:

| Name | SSZ equivalent | Description |
| - | - | - |
| `KZGCommitment` | `Bytes48` | A KZG commitment |
| `KZGProof` | `Bytes48` | A KZG proof |
| `Blob` | `ByteVector[BYTES_PER_BLOB]` | A basic data blob |

## Constants

The following values are (non-configurable) constants used throughout the
specification.

| Name | Value |
| - | - |
| `BYTES_PER_BLOB` | `uint64(2**17)` (= 131,072) |
| `MAX_BLOB_COMMITMENTS_PER_BLOCK` | `uint64(2**12)` (= 4,096) |

## Containers

#### Modified `BidTrace`

*Note*: `BidTrace` is modified to include blob count and blob gas fields.

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
    num_tx: uint64
    block_number: uint64
    num_blobs: uint64
    blob_gas_used: uint64
    excess_blob_gas: uint64
```

#### Modified `ExecutionPayload` 

*Note*: `ExecutionPayload` is modified to include blob gas fields.

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
    blob_gas_used: uint64
    excess_blob_gas: uint64
```

#### New `BlobsBundle`

*Note*: `SubmitBlockRequest` is modified to the modified `ExecutionPayload`.

```python
class BlobsBundle(Container):
    commitments: List[KZGCommitment, MAX_BLOB_COMMITMENTS_PER_BLOCK]
    proofs: List[KZGProof, MAX_BLOB_COMMITMENTS_PER_BLOCK]
    blobs: List[Blob, MAX_BLOB_COMMITMENTS_PER_BLOCK]
```

#### Modified `SubmitBlockRequest`

*Note*: `SubmitBlockRequest` is modified to the modified `ExecutionPayload` and
the new `BlobsBundle`.

```python
class SubmitBlockRequest(Container):
    message: BidTrace
    execution_payload: ExecutionPayload
    blobs_bundle: BlobsBundle
    signature: BLSSignature
```

#### Modified `BuilderSubmitNewBlockRequest`

*Note*: `BuilderSubmitNewBlockRequest` is modified by replacing the `capella`
field with a new `deneb` field.

```python
class BuilderSubmitNewBlockRequest(Container):
     deneb: SubmitBlockRequest
```
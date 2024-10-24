# Capella -- Relay Specs

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

This document presents the `mev-boost` relay specification for Capella. 

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
| `WithdrawalIndex` | `uint64` | an index of a `Withdrawal` |

## Constants

The following values are (non-configurable) constants used throughout the
specification.

| Name | Value |
| - | - |
| `MAX_WITHDRAWALS_PER_PAYLOAD` | `uint64(2**4)` (= 16) |

## Containers

#### New `Withdrawal`

```python
class Withdrawal(Container):
    index: WithdrawalIndex
    validator_index: ValidatorIndex
    address: ExecutionAddress
    amount: Gwei
```

#### Modified `ExecutionPayload` 

*Note*: `ExecutionPayload` is modified to include withdrawals.

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

#### Modified `SubmitBlockRequest`

*Note*: `SubmitBlockRequest` is modified to the modified `ExecutionPayload`.

```python
class SubmitBlockRequest(Container):
    message: BidTrace
    execution_payload: ExecutionPayload
    signature: BLSSignature
```

#### Modified `BuilderSubmitNewBlockRequest`

*Note*: `BuilderSubmitNewBlockRequest` is modified by replacing the `bellatrix`
field with a new `capella` field.

```python
class BuilderSubmitNewBlockRequest(Container):
     capella: SubmitBlockRequest
```
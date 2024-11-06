# Electra -- Relay Specs

## Table of contents

<!-- TOC -->
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Introduction](#introduction)
- [Constants](#constants)
- [Containers](#containers)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
<!-- /TOC -->

## Introduction

This document presents the `mev-boost` relay specification for Electra.

[`mev-boost`](https://boost.flashbots.net/) is an out-of-protocol mechanism to
connect Ethereum Proof-of-Stake validators to the external block-building
market. The [validator-side](https://github.com/flashbots/mev-boost) API is
specified in the [builder specs](https://github.com/ethereum/builder-specs). The
[relay-side](https://github.com/flashbots/mev-boost-relay) API is specified here
in the relay specs.

## Constants

The following values are (non-configurable) constants used throughout the
specification.

| Name | Value |
| - | - |
| `MAX_DEPOSIT_REQUESTS_PER_PAYLOAD` | `uint64(2**13)` (= 8,192) | Maximum number of execution layer deposit requests in each payload |
| `MAX_WITHDRAWAL_REQUESTS_PER_PAYLOAD` | `uint64(2**4)` (= 16)| Maximum number of execution layer withdrawal requests in each payload |
| `MAX_CONSOLIDATION_REQUESTS_PER_PAYLOAD` | `uint64(1)` (= 1) | Maximum number of execution layer consolidation requests in each payload |

## Containers

#### New `DepositRequest`

```python
class DepositRequest(Container):
    pubkey: BLSPubkey
    withdrawal_credentials: Bytes32
    amount: Gwei
    signature: BLSSignature
    index: uint64
```

#### New `WithdrawalRequest`

```python
class WithdrawalRequest(Container):
    source_address: ExecutionAddress
    validator_pubkey: BLSPubkey
    amount: Gwei
```

#### New `ConsolidationRequest`

```python
class ConsolidationRequest(Container):
    source_address: ExecutionAddress
    source_pubkey: BLSPubkey
    target_pubkey: BLSPubkey
```

#### New `ExecutionRequests`

```python
class ExecutionRequests(Container):
    deposits: List[DepositRequest, MAX_DEPOSIT_REQUESTS_PER_PAYLOAD]
    withdrawals: List[WithdrawalRequest, MAX_WITHDRAWAL_REQUESTS_PER_PAYLOAD]
    consolidations: List[ConsolidationRequest, MAX_CONSOLIDATION_REQUESTS_PER_PAYLOAD]
```

#### Modified `SubmitBlockRequest`

*Note*: `SubmitBlockRequest` is modified to include execution requests.

```python
class SubmitBlockRequest(Container):
    message: BidTrace
    execution_payload: ExecutionPayload
    blobs_bundle: BlobsBundle
    execution_requests: ExecutionRequests
    signature: BLSSignature
```

#### Modified `SubmitBlockRequest`

*Note*: `SubmitBlockRequest` is modified to the modified `ExecutionPayload` and
the new `BlobsBundle`.

```python
class SubmitBlockRequest(Container):
    message: BidTrace
    execution_payload: ExecutionPayload
    blobs_bundle: BlobsBundle
    execution_requests: ExecutionRequests
    signature: BLSSignature
```

#### Modified `BuilderSubmitNewBlockRequest`

*Note*: `BuilderSubmitNewBlockRequest` is modified by replacing the `deneb`
field with a new `electra` field.

```python
class BuilderSubmitNewBlockRequest(Container):
     electra: SubmitBlockRequest
```
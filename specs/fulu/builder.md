

## Introduction

This document presents the `mev-boost` relay specification for Fulu.

[`mev-boost`](https://boost.flashbots.net/) is an out-of-protocol mechanism to
connect Ethereum Proof-of-Stake validators to the external block-building
market. The [validator-side](https://github.com/flashbots/mev-boost) API is
specified in the [builder specs](https://github.com/ethereum/builder-specs). The
[relay-side](https://github.com/flashbots/mev-boost-relay) API is specified here
in the relay specs.


## Containers

### Modified `BlobsBundle`

```python
class BlobsBundle(Container):
    commitments: List[KZGCommitment, MAX_BLOB_COMMITMENTS_PER_BLOCK]
    # [Modified in Fulu:EIP7594]
    proofs: List[KZGProof, FIELD_ELEMENTS_PER_EXT_BLOB * MAX_BLOB_COMMITMENTS_PER_BLOCK]
    blobs: List[Blob, MAX_BLOB_COMMITMENTS_PER_BLOCK]
```

### Modified `SubmitBlockRequest`

*Note*: `SubmitBlockRequest` is modified to include the update in `BlobsBundle`.

```python
class SubmitBlockRequest(Container):
    message: BidTrace
    execution_payload: ExecutionPayload
    # [Modified in Fulu:EIP7594]
    blobs_bundle: BlobsBundle
    execution_requests: ExecutionRequests
    signature: BLSSignature
```

#### Modified `BuilderSubmitNewBlockRequest`

*Note*: `BuilderSubmitNewBlockRequest` is modified by replacing the `bellatrix`
field with a new `fulu` field.

```python
class BuilderSubmitNewBlockRequest(Container):
     fulu: SubmitBlockRequest
```
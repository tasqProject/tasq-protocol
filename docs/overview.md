# Protocol overview

This document describes TasQ end to end: the actors, the life of a job, the
cryptographic primitives, and how settlement works. It is a map. Each mechanism
has its own file under `docs/mechanisms/`.

## The problem

Renting a GPU today means trusting the operator with your model and your data.
The operator can read the weights, read the inputs, copy the outputs, or return
a result that was never computed. TasQ removes that trust from the critical
path. The client keeps control of the data, and the result carries evidence
that it is correct.

## Actors

- **Client.** Submits a job and pays for it. Holds the keys to the input and
  checks the result.
- **Operator.** Runs a machine that offers CPU and GPU time to the market.
- **Coordinator.** Matches jobs to offers, relays encrypted payloads, and
  records receipts. It never holds a decryption key and never sees plaintext.
- **Verifiers.** Independent parties that audit a sample of jobs and challenge
  bad results. In mode R the redundant operators are the first verifiers.
- **Ledger.** The chain that settles payment, moves credits and tracks
  reputation.

## Life of a job

1. **Intent.** The client signs an intent: the mode, the model reference, a
   commitment to the input, a budget and a deadline. The intent is an EIP-712
   typed message under the domain `{ name: "TasQ", version: "1", chainId }`.
2. **Admission.** The scheduler checks the intent against open offers and the
   operator's reputation, and admits it to a machine that qualifies for the
   requested mode. See `docs/mechanisms/scheduler.md`.
3. **Key exchange.** The client and the chosen machine run a hybrid key
   exchange, X25519 combined with ML-KEM. In mode A the client releases the key
   only after it has verified the machine's attestation.
4. **Run.** The machine decrypts the input, runs the job, and encrypts the
   output under a key only the client can open.
5. **Receipt.** The machine returns the output together with a receipt: a hash
   of the inputs and outputs, the mode, the attestation or the proof, and a
   signature. Hashes use BLAKE3. Batches are committed with a Merkle tree so a
   single result can be proven to belong to a batch.
6. **Verification.** The client checks the receipt for the mode it asked for.
   Attestation for mode A, agreement plus audit for mode R, a proof for mode P.
7. **Settlement.** On a valid receipt the ledger releases payment to the
   operator and updates reputation. A failed check withholds payment and lowers
   the operator's score.

## Cryptographic primitives

- **Key exchange: X25519 + ML-KEM.** A hybrid, so the session stays private even
  if one of the two schemes is later broken. ML-KEM is a post-quantum key
  encapsulation mechanism. This is a choice of cryptographic primitive only. It
  does not imply any quantum hardware.
- **Hashing: BLAKE3.** Fast, parallel, used for input and output digests and as
  the leaf hash in Merkle commitments.
- **Commitments: Merkle trees.** A batch of items commits to one root. Any item
  has a short inclusion proof against that root.
- **Signatures: EIP-712.** Intents and receipts are typed, signed messages, so
  they can be checked both off chain and by a contract.

## Assurance modes in one line each

- **A, Attested.** Trust the hardware enclave, verified by remote attestation.
- **R, Redundant.** Trust agreement across independent machines, backed by
  hidden audits.
- **P, Proven.** Trust a short proof checked against a commitment to the input.

The three modes trade privacy, cost and the kind of work they fit. A client
picks per job. See each mechanism file for the details.

## What this overview leaves out

The exact admission rule parameters, the audit sampling rate, the reputation
curve and the fee schedule are set by the contracts and are not final. This
document describes the shape of the system, not the final constants.

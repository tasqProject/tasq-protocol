# TasQ Protocol

TasQ is a marketplace for AI compute where the person paying for a job keeps
control of their model and their data. A job can run on a machine the client
has never met, and the client can still check that the right code ran and that
the result is correct, without the machine operator ever seeing the input.

This repository holds the public protocol: an overview, the mechanisms behind
each assurance mode, the scheduling and reputation rules, and research notes.
It does not contain the coordinator or the node client. Those live in separate
repositories, and some parts stay private.

## What is here

- `docs/overview.md`: the protocol in one read.
- `docs/mechanisms/`: one file per mechanism (attestation, redundancy, proofs,
  scheduling, reputation).
- `docs/research/`: notes and short write-ups.

## Assurance modes

A job picks one of three modes, chosen by how much the client needs to trust
the machine it runs on.

- **Mode A, Attested.** Inputs, weights and outputs are decrypted only inside a
  hardware enclave. The enclave proves the exact runtime image before any key is
  released, so the operator never sees the data. Best for confidential inference
  and fine-tuning.
- **Mode R, Redundant.** The job runs on several independent machines and the
  results must agree. A sample of the work is re-run as a hidden audit in mode A.
  The operators see the input, so this mode is for work that is not sensitive.
- **Mode P, Proven.** The machine returns a proof that the stated computation
  produced the stated output, checked against a commitment to the input. Best
  for small models and deterministic jobs.

## Where this runs

Settlement, credits and reputation live on Robinhood Chain. Jobs are described
by signed intents and matched to offers in an open market. The yellow paper
covers the threat model, the admission rule and what is not solved yet.

## Status

Pre-launch. The mechanisms here are described at the level of the yellow paper.
Numbers and parameters are not final until the contracts are deployed.

## License

Documentation and specifications in this repository are released under the
Apache License 2.0. See `LICENSE`.

Copyright The TasQ Project.

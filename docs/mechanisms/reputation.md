# Reputation

Reputation is a score per operator that the scheduler reads to decide which
machines may take which modes. It rises with settled jobs and falls with
rejected receipts and failed audits. A high score earns more work and access to
higher value jobs. A fault is expensive and slow to recover from.

## What it is

One number per operator, scaled so 1.0 is a long, clean record. It is kept on
chain so anyone can read it and the market can price it. The score is public;
the events that moved it are public.

## What moves it

- **A settled job** raises the score by a small amount. Returns diminish, so a
  machine cannot farm trivial jobs to a perfect score.
- **A rejected receipt** lowers it. The receipt failed verification for its mode,
  so no payment was made and the operator is marked.
- **A failed audit** lowers it sharply. In mode R a hidden audit re-ran the work
  in mode A and the operator's result did not match the ground truth. This is the
  strongest negative signal, because it is direct evidence of a wrong result.
- **Decay.** The score drifts toward a neutral baseline over time, so a machine
  that stops working loses standing and a past fault fades slowly rather than
  never.

## Thresholds per mode

Each mode has a minimum score to serve it.

- Mode A has the highest bar, because a client is trusting the machine with
  confidential work behind an attestation.
- Mode R has a middle bar, backed by the quorum and audits.
- Mode P leans on the proof rather than the operator, so its bar is the lowest.

A new machine starts below the mode A threshold and earns its way up through
mode R and mode P work. This gives the network a cheap way to test a machine
before trusting it with confidential jobs.

## Why on chain

Keeping reputation on chain means the score cannot be quietly rewritten, the
history is auditable, and a client can verify an operator's standing without
trusting the coordinator. The settlement contract and the audit coordinator are
the only callers allowed to move a score. See `tasq-contracts`.

## What this leaves to the contracts

The exact increment, the penalty sizes, the decay rate and the per mode
thresholds are governance parameters. This document fixes their direction, not
their final values.

## Related

- `docs/mechanisms/scheduler.md`, which reads these thresholds.
- `docs/mechanisms/redundancy.md`, where failed audits come from.

# Scheduling and admission

The scheduler decides which machine runs a job. It matches an intent to offers,
enforces that the machine qualifies for the requested mode, and for mode R it
applies the admission rule that keeps cheating unprofitable.

## Inputs

- The signed intent: mode, model, input commitment, budget and deadline.
- Open offers from the market: GPU class, modes supported, region, price and the
  operator's reputation.
- Protocol parameters: the audit rate, the stake per seat and the slash fraction.

## Matching

A machine is a candidate for an intent only if all of the following hold.

- It advertises the requested mode. Mode A needs an attestable GPU and a CPU TEE.
- Its reputation clears the threshold for that mode. See
  `docs/mechanisms/reputation.md`.
- Its price is within the intent's budget and it can meet the deadline.

Candidates are ranked by price, then reputation, then recent uptime. The lowest
admissible candidate wins.

## Independence, for mode R

Mode R only means something if the machines are independent. The scheduler
places the r seats of a unit on machines that differ by operator, and where
supply allows, by region and by hardware. Two seats never land on the same
operator. Without this, a quorum is weaker than it looks and collusion is
cheaper.

## The admission rule, for mode R

Mode R accepts a unit only if an adversary that captures the quorum would lose
more in slashed stake, in expectation, than it could gain by returning a wrong
result. That is a cap on the value of a single unit for a given audit rate.

- Raising the audit rate raises the cap, because cheating is caught more often.
- Raising the quorum raises the cap, because more machines must be captured.
- Raising the stake per seat raises the cap, because a caught fault costs more.

If an intent's declared unit value is above the cap at the current audit rate,
the scheduler either raises the audit rate for that job or refuses mode R and
tells the client to use mode A. The exact function and its constants are set by
the contracts and are not fixed here. The app exposes the same relationship in
its pricing preview.

## Placement and fan out

Once a unit is admitted, the scheduler fans it out to its seats, holds the
budget in escrow through settlement, and records which machines were chosen so a
later audit can re-run a sample on an independent, attested machine.

## What this leaves to the contracts

The admission constants, the audit sampling rate and the ranking weights are
governance parameters. This document fixes their shape and their direction, not
their final values.

## Related

- `docs/mechanisms/redundancy.md` for how mode R uses the quorum and audits.
- `docs/mechanisms/reputation.md` for the thresholds the matcher reads.

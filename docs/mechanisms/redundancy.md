# Mode R: redundant execution

Mode R is for work that is not sensitive but still needs to be correct: batch
inference, embeddings, and data jobs where the operators may see the input. The
job runs on several independent machines and the results must agree. A sample of
the work is re-run as a hidden audit in mode A.

## Goal

Give the client a guarantee of the form: "this result was produced by q
independent machines that agreed, and a random sample of such jobs is re-checked
in an attested enclave, so cheating is caught and punished." Privacy is not a
goal here. The operators see the input.

## Parameters

A mode R job carries two numbers:

- `r`, the number of machines that run the job.
- `q`, the number that must return the same result for it to settle. `q <= r`.

A common choice is q of r with q a strict majority, so a single bad machine
cannot change the answer. The exact default is set by the scheduler and the
client can raise it.

## Flow

1. **Fan out.** The scheduler places the job on `r` independent machines. They
   must be independent by operator and, where possible, by region and hardware,
   so they do not share a failure or a collusion path.
2. **Run.** Each machine runs the job and returns an output with a BLAKE3 digest
   and a signature.
3. **Agree.** The coordinator groups the results by output digest. If at least
   `q` machines share a digest, that result settles. If no group reaches `q`, the
   job fails and no one is paid for a wrong answer.
4. **Pay.** The agreeing machines are paid. Machines in a losing group are not,
   and the disagreement is recorded against them.

## Hidden audits

Agreement alone can be gamed if machines collude. Mode R defends against this
with audits the operators cannot predict:

- A fraction of mode R jobs, chosen at random, are silently re-run in mode A on
  an attested machine.
- The attested result is the ground truth. Any machine whose mode R output
  disagrees with the audit is slashed, even if it was in the agreeing group.
- Because a machine cannot tell an audited job from a normal one, the safe
  strategy is to always compute honestly.

The audit rate trades cost against deterrence. It is set so that the expected
penalty for cheating outweighs the saved compute. The exact rate is a scheduler
parameter and is not fixed here.

## Threat notes

- **A single dishonest machine.** Outvoted by the quorum, then caught by audits
  over time.
- **Collusion across the quorum.** The audit re-runs in mode A on a machine
  outside the colluding set, so a shared lie still fails the ground truth check.
- **Independence failure.** If machines are not actually independent, the quorum
  is weaker than it looks. The scheduler enforces independence by operator and,
  where it can, by region and hardware. See `docs/mechanisms/scheduler.md`.

## Cost

A mode R job pays for `r` runs plus a share of the audit cost. It is cheaper
than mode A per unit of trust when the data is not sensitive and the model is
large, since attestation overhead is avoided on the `r` runs.

## Related

- `docs/mechanisms/attestation.md`, which mode R uses as its audit.
- `docs/mechanisms/reputation.md`, for how disagreement changes an operator's
  score.

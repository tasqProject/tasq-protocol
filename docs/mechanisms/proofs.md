# Mode P: proven execution

Mode P is for work where the result can carry its own proof: small models and
deterministic jobs. The machine returns a proof that the stated computation
produced the stated output, checked against a commitment to the input. The
verifier learns nothing about the input beyond the commitment, but the machine
that ran the job did see it.

## Goal

Give the client a guarantee of the form: "this output is the correct result of
running this program on an input that matches my commitment." The client, or
anyone, can check the proof without re-running the work and without seeing the
input.

## What is proven

A mode P statement ties three things together:

- The program, identified by a hash of its compiled form.
- The input, bound by its commitment, the BLAKE3 digest in the intent.
- The output, bound by its commitment.

The proof shows that running the program on an input with that commitment yields
an output with that commitment. It is a succinct proof, a zk-STARK, so checking
it is far cheaper than redoing the computation.

## Flow

1. **Commit.** The client includes the input commitment in the signed intent.
2. **Run and prove.** The machine runs the program and, alongside the output,
   produces a proof for the statement above.
3. **Return.** The receipt carries the output, its commitment, and the proof.
4. **Verify.** The client checks the proof against the program hash and the two
   commitments. A valid proof settles the job. See `tasq-contracts` for the
   on-chain verifier interface.

## Why STARKs

- No trusted setup, so there is no toxic waste and nothing to leak.
- Transparent and post-quantum in its assumptions, it rests on hash functions
  rather than pairings. As with the rest of TasQ, this is a choice of primitive
  and does not imply any quantum hardware.
- Succinct verification, so a contract or a laptop can check a proof quickly.

The cost is on the prover side: producing the proof is heavier than the bare
computation. That is why mode P fits small models and deterministic jobs, where
the proving overhead is acceptable, rather than large model inference.

## Limits

- **Privacy of the input.** Mode P proves correctness, not privacy. The machine
  that runs the job sees the input. A client that needs the input hidden from the
  machine uses mode A.
- **Non determinism.** Floating point and parallel reductions can make a model
  non deterministic across hardware. Mode P requires a deterministic program, so
  models are compiled to a fixed, reproducible form before they qualify.

## Related

- `docs/mechanisms/attestation.md` for mode A, which hides the input from the
  machine.
- `docs/overview.md` for how commitments and receipts fit the job lifecycle.

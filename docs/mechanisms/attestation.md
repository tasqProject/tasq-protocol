# Mode A: attested execution

Mode A is for work where the operator must never see the data: confidential
inference and fine-tuning. The job runs inside a hardware enclave, and the
client releases the decryption key only after the enclave has proven what code
it is about to run.

## Goal

Give the client a guarantee of the form: "my input was decrypted only inside an
enclave running this exact image, on genuine hardware, and the output came back
to me encrypted." The operator that owns the machine is outside this boundary
and learns nothing about the plaintext.

## Trust boundary

The enclave is the trust boundary. TasQ targets confidential computing on the
GPU, with the CPU side in a trusted domain:

- GPU: NVIDIA Confidential Computing, which keeps model weights and activations
  encrypted in GPU memory and attests the GPU state.
- CPU: a trusted execution environment such as Intel TDX for the host process
  that feeds the GPU.

Everything outside the enclave, the operator, the host operating system, the
coordinator and the network, is treated as untrusted.

## Flow

1. **Offer.** The operator advertises a machine that supports mode A. The offer
   names the platform and the measurement roots the client can expect.
2. **Intent.** The client submits a mode A intent with a commitment to the
   input and the reference of the runtime image it requires.
3. **Attestation.** The enclave produces a remote attestation: a signed
   quote over its measurements, including the runtime image, bound to a fresh
   public key the enclave generated inside the boundary.
4. **Verification.** The client checks the quote against the hardware vendor's
   roots of trust, checks that the measured image matches what it asked for, and
   checks freshness with its own nonce. See "What the client checks" below.
5. **Key release.** Only after the quote verifies does the client complete the
   X25519 plus ML-KEM exchange against the enclave's attested key. The session
   key now exists only inside the enclave and on the client.
6. **Run.** The enclave decrypts the input, runs the job, and encrypts the
   output under the client's key.
7. **Receipt.** The machine returns the output with a receipt: BLAKE3 digests of
   the input and output, the attestation, the mode, and a signature. The client
   stores the receipt as evidence.

## What the client checks

- The quote is signed by a key that chains to the hardware vendor's root.
- The measured runtime image equals the reference in the intent.
- The quote is bound to the client's nonce, so it cannot be replayed.
- The attested key is the key used in the session, so a valid quote cannot be
  stapled to a different, attacker held key.

If any check fails, the client never releases the key, so the input is never
exposed.

## Threat notes

- **Malicious operator.** Sees only ciphertext and attestation material. It
  cannot read the input or forge a passing quote for a different image.
- **Replay.** The client nonce and the fresh in-enclave key make a captured
  quote useless later.
- **Rollback or swapped image.** A different image produces a different
  measurement, so verification fails.
- **Side channels.** Out of scope for the protocol and handled, as far as they
  can be, by the hardware platform and the enclave image. TasQ does not claim to
  defend against a hardware vendor compromise of its own root of trust.

## Cost

Mode A adds the overhead of attestation and encrypted execution on top of
ordinary GPU time. The premium is small relative to the GPU time itself. The
exact figure depends on the platform and is published with the market offers,
not fixed here.

## Related

- `docs/overview.md` for the job lifecycle.
- `docs/mechanisms/redundancy.md` for mode R, which uses mode A as its audit.

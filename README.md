# Fastmint Guard

A safety-guarded launcher for timed first-come-first-served free mints on BNB
Chain. It exists because of one rule: **simulate before broadcast, and never
sign a call you haven't checked.**

Free-mint pools are small and drain in under a minute. The temptation is to
fire blind and fast. This tool is the opposite: all verification happens
*before* the open, and at the open second it fires exactly once — through the
wallet's own preview — with hard safety checks that can still stop it.

## How it works

1. **Recon (`recon.js`)** — before the open: verify the target contract, the
   free-mint eligibility, the wallet session, and pre-build the calldata.
2. **Fire (`fire.js`)** — pre-staged before the open. From two seconds before
   the open it spins `eth_call` across six public BSC RPCs (latest + pending,
   every 120 ms). On the first successful simulation it runs the wallet
   preview and, only if the preview is clean, executes exactly once.
3. **Safety gates (every fire)** — the preview must show simulation success,
   no allowance or authority changes, no risk flags, and native outflow under
   the cap. A lock file blocks re-runs. On any execute error it never
   retries. Value above zero requires an explicit `--maxValue`.

## What it deliberately does NOT do

- Hold keys. Signing stays inside the Binance Agentic Wallet CLI (`baw`);
  this repo contains no key material, no seeds, no API keys.
- Blind-sign. Every broadcast goes through the wallet's preview first.
- Retry broadcasts. One attempt, one result, honestly reported.

## Try the simulator (no wallet needed)

The read-only half of the guard runs as a public endpoint — send a mint call,
get a verdict, sign nothing:

- `POST /simulate` with `{to, from, value: "0", inputData}` → `eth_call`
  simulation plus a safety verdict (`SIMULATION_OK` or `REFUSE`).
- `GET /` for usage, `GET /health` for status.

[ENDPOINT URL — added after deployment]

## Honest limits

The wallet's preview-plus-execute round trip takes ~4–5 seconds on
Binance's side and cannot be removed. For a tiny pool raced by direct-key
bots, even a tight flow can lose. This tool makes the attempt as fast as the
safety rails allow — it does not promise a win.

## Files

- `recon.js` — pre-open verification and calldata prep
- `fire.js` — staged launcher with the safety gates
- `lib.js` — shared RPC / simulation helpers
- `recon.sh` — recon wrapper

Built for the zkVerify Agent Games (Wildcard track). The same idea —
decisions constrained by verification, not described by logs — is explored
cryptographically in the companion Creator Track article, which generates a
real Groth16 proof with the tournament's gatekeeper circuit.

# Attestant

A verifier for autonomous work. It takes what an agent says it did, derives the same facts independently from the systems of record, and reports every disagreement as a finding.

Status: design stage. The interface below is the design, not a shipped API.

## Why

Asking an agent to log its own actions is asking the witness to grade its own testimony. The fields an agent writes about itself are the first thing an adversarial reader attacks: a cost attribute the agent set can be zero, and "escalated to a human: true" written by the agent is not evidence that a human was involved.

The only thing worth treating as evidence is the execution record, and the execution record lives in systems the agent does not write.

## What it derives, and from where

| Claim | Declared by the agent | Derived by attestant from |
|---|---|---|
| Commands run and their outcomes | span attributes | process exit codes, git objects, the workspace diff |
| Tests passed | summary text | the CI provider's own result for that commit |
| Cost | span attribute | token counts multiplied by a dated pricing table |
| Approval granted | boolean in the run log | the approval store's record, keyed by the action hash |
| External effect | narration | the target system's record (ticket, payment, commit, message id) |
| Files changed | narration | content addressed comparison of the workspace |

## Principles

1. Every field in an evidence pack is labelled `derived` or `declared`. A pack with no derived fields is not evidence.
2. A declared fact that disagrees with a derived fact is a finding, never a correction. Picking a winner silently destroys the value of the record.
3. No model in the verification path. A verifier that can be persuaded is not a verifier.
4. Read only from sources the agent cannot write. If a source is writable by the agent, it is not a system of record.
5. Offline by default. Verification of a completed run should not require a network beyond the systems of record themselves.

## Shape of the API

    attestant verify --run <run-id> --sources git,ci,pricing,approvals --out pack.json

Output is a pack of claims, each with `claimed`, `derived`, `source`, `status` in `match`, `mismatch` or `unverifiable`, and a signature over the canonical serialisation.

## Roadmap

- Derivers for git, filesystem, GitHub Actions and JUnit XML output.
- Pricing table with a dated snapshot, so cost can be recomputed months later.
- Signed packs, hash chained, with a replay command.
- Approval binding: verify that the action executed is the action that was approved.

## What this is not

Not an observability product. Observability helps you debug. This decides whether a claim is true.

## License

MIT, see [LICENSE](LICENSE).

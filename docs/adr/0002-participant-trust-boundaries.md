# ADR 0002: Separate participant inputs, outputs, and authority

**Status:** Proposed

## Context

A future workflow may feed one participant's answer to another for blind review and voting. A well-formed answer can still contain misleading or instruction-like text, and a useful audit record can contain information a reviewer shouldn't see. [ADR 0001](0001-stateful-cli.md) separates observation from intervention; this proposal describes how to preserve that boundary between participants.

## Proposal

- **Treat model output as evidence, not authority.** Admit proposals and ballots only after deterministic validation. Validating the shape of a submission doesn't make its prose trustworthy. Neither a proposal nor a ratified resolution authorizes execution by itself.
- **Separate review data from diagnostics.** Preserve raw submissions and provenance for authorized operators, but build the blind view from explicitly permitted fields. Use fresh sessions for blind phases and resume conversations only where continuity is intended. A blind view hides recorded authorship, not clues in the text itself, and reviewers must not be able to read the raw records directly.
- **Normalize proposals only when needed.** An optional LLM adapter could extract a structured proposal from free-form text, without tools or author metadata. Code would validate the result; the adapter could decline to extract anything when the source is ambiguous. Keep the original for comparison and freeze the candidate set and the exact text reviewers see before voting. Require strict ballots referring to those frozen candidates; don't use another model to infer a voter's intent.
- **Share task inputs, not writable workspaces or credentials.** When supplied, snapshot `AGENTS.md` for the run and deliver it through each adapter as appropriate. Isolate participants' writable workspaces when they need them. For direct APIs, resolve credentials by environment variable name, optionally from an explicit `.env` file. Let external harnesses manage their own login. Don't persist credential values in run records.

## Consequences and open questions

Schema validation and blind views reduce what crosses the boundary; they don't prevent a reviewer from being influenced by the content it sees. The authorization model, filesystem isolation, and normalization criteria still need to be defined. Blind review, normalization, and voting are not prerequisites for the first multiplexer.

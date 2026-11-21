# ADR 0001: Start with a stateful CLI

**Status:** Accepted

## Context

An agent should be able to ask Conclave to convene a panel, wait for the result, and later find out how it was reached. The first workflow collects independent answers. Later workflows may collect proposals, review them blindly, and vote. Callers shouldn't have to manage participant conversations or coordinate those phases themselves.

## Decision

Use a **local, stateful CLI** as the first interface. The initial delegation call blocks until the run reaches an outcome. Conclave persists the run and returns its ID so it can be inspected after the process exits. The CLI exposes machine-readable results so a harness can distinguish a partial result or a lack of consensus from a command failure without interpreting prose.

The interface has three operations:

- **Delegate:** Start a workflow with a selected panel. Asking for independent answers and asking for a vote are different workflows, not different integration models.
- **Observe:** Inspect the outcome and why the run advanced or stopped. A blind review view and privileged diagnostics are two views of the same run, not separate operations. A run ID identifies a run; it doesn't grant access to its raw records.
- **Intervene:** Explicitly change what happens next when the workflow permits it, for example by cancelling, clarifying, or approving a step. Defer a dedicated intervention interface until a workflow needs it. Participant output cannot authorize an intervention or execution.

The first release implements **delegate** and **observe** for named panels, with concurrent fan-out, per-participant timeouts, and an inspectable run record. Treat multiplexing as a one-phase workflow with an explicit rule for when it closes. Completion isn't the same as successful submission: timeouts and failures can close out a participant without discarding the results that did arrive. Later phases can enforce quorum or return no decision. Conclave owns the record of what was sent and received rather than relying on session IDs maintained by external harnesses.

## Alternatives considered

- **Stateless CLI:** Easy to invoke, but the caller would have to preserve results and reconstruct provenance or phase history.
- **Library, harness plugin, or MCP server first:** Can offer a more native integration, but makes the first caller depend on a particular runtime or tool protocol. These can be added around the same operations when needed.
- **HTTP service or daemon first:** Useful for remote access or runs that outlive their callers, but unnecessary for the initial blocking workflow.

## Consequences

Shell-capable harnesses can use Conclave without an SDK or a server. Persistence adds a storage obligation, and blind review requires controlling access to raw records; a CLI view alone is not a security boundary. Exact commands, wire schemas, exit codes, storage technology, and diagnostic access controls remain separate decisions.

Proposal rounds, ranked-choice methods, and governance variants such as an LLM Council can build on the same run model. [ADR 0002](0002-participant-trust-boundaries.md) proposes boundaries for participant inputs and outputs.

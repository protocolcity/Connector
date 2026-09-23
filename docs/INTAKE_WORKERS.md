# Proposed external intake workers

An intake adapter translates an authorized external event into scoped work.
It is not a general permission to read accounts, send messages or execute projects.
This repository does not ship an intake worker.

The adapter contract should state source access, project destination, signing
identity, allowed operations, schedule/trigger, budget, deduplication key,
retention, diagnostics and stop conditions. Register execution with WorkForce
only after the selected deployment verifies that capability. WorkLane owns the
resulting work records and guards; source event IDs are evidence, not another
work database.

Use stable event IDs and idempotent writes, test repeated/out-of-order delivery,
retain source provenance, and prevent a work-order update from feeding back into
a new source event indefinitely. Distinguish empty, unauthorized, unavailable and
rate-limited input. Use bounded retries and preserve the last successful cursor
without inventing progress.

Default tests use synthetic events and disposable stores. A live trial needs
actual source authorization and a bounded destination. Reading an email does not
authorize sending or replying to it. Keep account data and credentials outside
public source and redact diagnostics by default.

See [architecture](../ARCHITECTURE.md) and [prerequisites](SEQUENCING.md).

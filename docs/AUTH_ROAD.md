# Proposed authentication boundary

No token service, localhost bypass, network authorization middleware or revocation
mechanism is established by this paper. Each engine must implement and test its
own boundary before a remote client can rely on it.

A future connection needs authenticated transport and explicit server, workspace,
client and project identities. Authorization limits operations independently of
which provider supplies inference. Configuration flags, caller-supplied author
names and signed-looking comments are attribution, not authentication.

The implementation must define credential issuance, expiry, rotation, revocation,
storage and audit behavior. Tests must cover invalid/expired credentials, replay,
cross-project access, wrong-host responses, reconnect and mid-run revocation.
A trusted-network label alone must not silently grant write authority.

Credentials belong in the deployment's protected store and are referenced from a
connection record. Never put real tokens in public templates, work checkpoints,
ordinary logs or screenshots. Sanitized diagnostics should identify a failed
check without returning credential values.

WorkLane still owns work-state guards. WorkForce still owns executor permissions,
process and capacity controls. Connector may describe and negotiate capabilities;
it must not silently override either engine's refusal. BP displays the observed
result and source time.

Follow [SEQUENCING.md](SEQUENCING.md). Exact protocols, middleware and configuration
names remain implementation decisions until a tested adapter documents them.

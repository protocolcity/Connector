# Connector implementation prerequisites

Status: design only. Historical gate numbers and dated host milestones are not
current readiness evidence. Local BP/WorkLane/WorkForce operation does not depend
on Connector shipping.

Before remote-join implementation is accepted:

1. Define the supported hosts, threat model, client identities and scopes. Name
   the exact engine versions and transport involved.
2. Implement authentication and authorization in each owning engine. Test absent,
   invalid, expired and revoked credentials and cross-workspace/cross-project refusal.
3. Verify host/workspace identity during discovery and connection. Never substitute
   another endpoint after a failed connection.
4. Define protected connection storage, credential references and redacted support
   evidence. Secrets stay outside source and ordinary connection records.
5. Demonstrate a join/disconnect/reconnect journey in isolated environments with
   truthful unavailable and partial states.
6. For remote execution, verify artifact access, source/instruction revisions,
   exclusive ownership, stopped-writer evidence, retries and recovery across hosts.
7. Publish working versioned installation instructions only after these checks.
   Documentation examples must execute against the named implementation.

A work order may implement a bounded prerequisite within existing authorization.
This list does not require another plan-approval ceremony. Actual credential,
network, publication or deployment decisions remain with the selected workspace's
applicable authority. File cross-engine changes in each owning product.

External intake is a separate adapter: scope access to its source, establish
stable event identity, test duplicate delivery and prevent feedback loops before
allowing writes into WorkLane. Connecting a mailbox is not permission to send mail.

Retired project-chat/Nostr-room prototypes are historical designs. No current
join or execution flow requires them.

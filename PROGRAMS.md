# Connector design areas

| Area | Current state | Next evidence needed |
|---|---|---|
| Workspace join | Proposed | Tested host identity, scoped discovery and connection lifecycle |
| Engine authentication | Proposed | Owning-engine threat model, enforcement, expiry and revocation tests |
| Remote execution | Proposed | Authenticated runner, artifact transport, process ownership and recovery evidence |
| External intake | Proposed | Source-specific authorization, identity, deduplication and rate-limit tests |

No row grants credentials, publication or deployment authority. Follow
[SEQUENCING.md](docs/SEQUENCING.md) and the selected workspace's actual work orders.

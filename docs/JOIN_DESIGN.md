# Proposed workspace join experience

This is a design, not a setup guide for a shipping remote adapter.

The person selects a workspace and execution host, sees what access is requested,
authenticates through that host's supported method, and verifies the returned
workspace identity. A successful join shows the granted project scope and usable
capabilities with their evidence time. It does not automatically register or hire
an agent, claim work, or start execution.

The connection record references credentials held in protected local storage.
The client must not print tokens into ordinary logs, work orders or support bundles.
A disconnect retains work history. Revocation prevents subsequent authorized
operations at the owning engine; a UI label alone cannot enforce it.

## States the interface must distinguish

| State | User meaning |
|---|---|
| Not configured | No connection has been selected |
| Connecting | Identity and transport verification is in progress |
| Authentication required | This endpoint requires a supported credential action |
| Connected | The endpoint and explicitly granted capabilities were observed |
| Partial | Some configured engines or capabilities are unavailable |
| Stale | Previous evidence is retained but must be refreshed before action |
| Refused | Identity, scope or authorization did not match |
| Disconnected | No active connection is being used; durable work remains |

Show the next meaningful action and preserve the person's work after errors.
Do not convert an unreachable host into an empty project list, infer liveness
from GitHub, or suggest switching providers to bypass a permission refusal.

See [architecture](../ARCHITECTURE.md), [auth boundaries](AUTH_ROAD.md),
[connection record](JOIN_CONFIG_TEMPLATE.md) and [prerequisites](SEQUENCING.md).

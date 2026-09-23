# Proposed connection record

This example is a design artifact. It is not accepted configuration for current
WorkLane, WorkForce or an available Connector CLI. Do not paste it into a provider's
MCP settings and assume it creates network access.

```json
{
  "schema": "connector.connection-proposal/v1",
  "workspace_id": "example-workspace",
  "host_id": "example-host",
  "endpoint": "https://workspace.example.invalid",
  "project_scope": ["example-project"],
  "requested_capabilities": ["work.read"],
  "credential_reference": "protected-store-reference"
}
```

The future adapter must verify these claims against its authenticated endpoint.
Granted capabilities and their observation/expiry times belong in a separate
connection receipt. No secret value belongs here. A host must not be inferred
from a fixed port, folder name or provider name.

For supported local MCP installation, use the current
[WorkLane installation guide](https://github.com/protocolcity/WorkLane/blob/main/INSTALL.md).
For a local operations UI, use
[BluePrint deployment instructions](https://github.com/protocolcity/BluePrint/blob/main/docs/operations/DEPLOYMENT.md).
These existing local products do not establish the remote proposal above.

See [join design](JOIN_DESIGN.md), [authentication boundary](AUTH_ROAD.md) and
[implementation prerequisites](SEQUENCING.md).

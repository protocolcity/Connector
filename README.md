# Connector

Connector is the proposed join layer for working with a workspace from another
host. This repository is a design reference. There is no installable Connector
package, production service or verified network join command here.

A local BluePrint workspace works without Connector. BluePrint shows operations,
WorkLane owns work records, and WorkForce controls execution. Calling a hosted
AI model from a local CLI does not make the workspace remotely accessible.

Start with [architecture](ARCHITECTURE.md), [implementation prerequisites](docs/SEQUENCING.md),
[join experience](docs/JOIN_DESIGN.md), [authentication boundaries](docs/AUTH_ROAD.md)
and the [connection record template](docs/JOIN_CONFIG_TEMPLATE.md).
[Integration workers](docs/INTAKE_WORKERS.md) describes a separate proposed intake pattern.

For working local software, follow the current documentation in
[BluePrint](https://github.com/protocolcity/BluePrint),
[WorkLane](https://github.com/protocolcity/WorkLane) and
[WorkForce](https://github.com/protocolcity/WorkForce).
Do not use historical split-service URLs or proposed environment variables as a
remote setup recipe.

[PROGRAMS.md](PROGRAMS.md) summarizes the design areas. Earlier project-chat and
identity experiments are retired; their short references point to current
boundaries. Their older revisions remain in Git history, not in the active setup path.

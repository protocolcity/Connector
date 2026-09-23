# Connector architecture

Status: proposed remote-join architecture. No production adapter is implemented
in this repository. The supported local suite is documented by its owning products.

## One owner per responsibility

| Responsibility | Owner |
|---|---|
| Work orders, claims, gates, checkpoint records and write validation | WorkLane |
| Registered executors, process control, provider capability, quota and run evidence | WorkForce |
| Operations views and supported actions | BluePrint |
| Join experience, connection description and host-selection design | Connector |
| Authentication and authorization at a network endpoint | The engine serving that endpoint |
| Credentials and host configuration | The deployment, outside distributable source |
| Project code and domain data | The independent project |

A future client selects an explicit workspace and host, verifies the endpoint's
identity, authenticates, and obtains scoped capabilities. It then calls the owning
engine. Connector must not create a second work-order database or bypass an
engine's claim, permission or capacity checks.

## Required distinctions

A provider, model, runner, account pool, worker identity and execution host are
different facts. A configuration file does not prove authentication or liveness.
Repository events are delivery evidence. A hosted-model API call is not a remote
workspace adapter. Moving work between providers does not export a proprietary
conversation; a receiving runner validates a durable checkpoint and artifacts.

Network handoff also needs transport, artifact availability and stopped-writer
proof on the original host. Local process/lock checks cannot attest an arbitrary
remote process. An unreachable host is unknown, not safely stopped.

## Repository structure

AGENTS.md holds contributor instructions. This file states boundaries. docs/
holds proposed join/auth/intake interfaces and prerequisites. PROGRAMS.md is a
design index. Provider pointers refer to AGENTS.md. Runtime rosters, deployment
receipts, private research and credentials are not product source.

Retired in-app project chat and Nostr-room experiments are not prerequisites for
remote work coordination. They must not be restored by following an old paper.

Changes to ownership update this file and the affected engine contract together.
A design milestone is complete only as a design; it is not shipped behavior.

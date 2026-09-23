# Connector product instructions

Connector defines joining a workspace from another execution host. This repository
currently contains designs and templates, not an installable service or tested
remote adapter. Do not infer runtime capability from a design document.

Read [ARCHITECTURE.md](ARCHITECTURE.md) for ownership and
[docs/SEQUENCING.md](docs/SEQUENCING.md) for implementation prerequisites.
WorkLane owns work records and write authority; WorkForce owns execution,
registered workers and capacity; BluePrint presents their verified state.
Connector owns join experience and connection metadata. Each engine must enforce
its own authentication and authorization.

Keep host instructions, rosters, credentials, customer data and operational
history outside public source. Use synthetic examples and explicit capability
status. Provider pointers refer to this file. Read the selected workspace's
instructions for authorization and discover its actual registered executor;
historical seat names do not establish capacity.

Documentation changes may clarify proposals. They do not enable network access,
mint credentials, register workers or deploy services. Any implementation needs
an owning work order, explicit scope, a threat model and tests of the prerequisites.
Cross-component changes belong to their owning repositories and work orders.
Preserve existing work and history; publish only reviewed product material under
the selected workspace's authorization.

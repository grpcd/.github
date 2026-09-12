# Deployment

## Regional clusters

Each geographic region runs its own grpcd cluster with its own storage backend.
Nothing is synchronized across regions. Services register in their region and
discover in their region, so discovery stays local and a regional failure stays
regional.

```
┌───────────────────────┐         ┌───────────────────────┐
│    us-east-1          │         │    eu-west-1          │
│                       │         │                       │
│  regional DNS name    │         │  regional DNS name    │
│        │              │         │        │              │
│  ┌─────┴──────┬────┐  │         │  ┌─────┴──────┬────┐  │
│  ▼            ▼    ▼  │         │  ▼            ▼    ▼  │
│ D-1          D-2  ... │         │ D-1          D-2  ... │
│  │            │    │  │         │  │            │    │  │
│  └─────┬──────┘    │  │         │  └─────┬──────┘    │  │
│        ▼           │  │         │        ▼           │  │
│   ┌──────────┐     │  │         │   ┌──────────┐     │  │
│   │  Redis   │     │  │         │   │  Redis   │     │  │
│   └──────────┘     │  │         │   └──────────┘     │  │
└───────────────────────┘         └───────────────────────┘
```

A regional DNS name resolves to that region's instances. Every instance answers
every lookup, since every fact it serves is in the shared storage backend, so
which instance a service or client lands on does not matter.

## What a service's environment carries

- `GRPCD_ADDRESS` — the regional cluster's name. Unset, the service serves
  without registering or discovering.
- `GRPC_SERVER_VERSION` — what the service's info endpoint reports.

## What the server's environment carries

- `GRPC_MAX_CONNECTION_AGE=0`. A registration lives as long as the stream
  holding it, and a GOAWAY ends that stream. At the default of ten minutes every
  registration in the mesh is torn down and rebuilt on that timer; it works, and
  it costs a reconnect per service per interval.
- `STORAGE_BACKEND` and `STORAGE_ADDRESS`, the shared store the region's
  instances write to. The server README lists the backends.

## Health probes

The server answers health for two entries. `""` says the process is alive.
`grpcd.GRPCDService` says whether the store can be reached. A balancer or
orchestrator that should route around an instance that has lost its store asks
for the second.

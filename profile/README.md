# grpcd

gRPC method discovery. A service opens a stream to grpcd naming the methods it
serves; the registration lives exactly as long as that stream. A service that
needs another asks grpcd for one of that service's methods and is handed an
address that serves it. There is no refresh interval, no deregistration call,
and no expiry: the evidence a service is up is the connection it holds.

## Further Reading

- [Ecosystem](../docs/ecosystem.md) — how a service is assembled and how it
  registers.
- [Deployment](../docs/deployment.md) — regional clusters and what a service's
  environment needs.

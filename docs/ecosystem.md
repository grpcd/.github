# Ecosystem

A service that takes part in grpcd is built from three layers, each a
separate module, each usable without the one above it.

## How a service serves

[grpc-foundation](https://github.com/sonic-original-software/grpc-foundation)
makes the decisions every gRPC server makes once: connection limits and
keepalive, panic recovery, OpenTelemetry wiring, a logger that reaches request
handlers carrying their trace, an error vocabulary, and a shutdown that stops
serving and then exports what it recorded.

## What a service exposes

[grpc-service](https://github.com/sonic-original-software/grpc-service) mounts
the endpoints every server in the mesh answers alongside the caller's own:
standard health and reflection, an info service reporting the version, and a
diagnostics service reporting the state of each dependency the caller names. Its
`Register` returns the fully qualified names of the caller's own methods, which
is the list grpcd advertises.

## How a service registers and discovers

[grpcd/client](https://github.com/grpcd/client) takes that list and holds the
registration stream open for the life of the process. Its `discover` package
supplies a resolver for a plain `*grpc.ClientConn`: it asks grpcd for an
upstream's method, probes the candidate from the service's own network
position, pushes the reachable one into the connection, and discovers again
when the transport drops. The generated client built on that connection never
sees an address change.

## The life of a registration

1. The service binds its listener and reads the port it received.
2. It opens `Register`, sending its name, its method list, and that port. grpcd
   reads the IP off the connection and composes the address.
3. grpcd writes one row per method and holds the stream.
4. The stream ends, by clean shutdown, crash, or transport failure. grpcd
   removes the rows it wrote.

A service that cannot reach grpcd serves anyway and registers when it can.

## Descriptors and health

Proto descriptors come from each service's reflection endpoint, and health from
each service's health endpoint. grpcd carries addresses only, so a client that
has an address goes to the service for everything else about it.

# Trace Context

## What is it?

We propose a set of HTTP headers which propagate a distributed trace, even when a request and its associated trace is served by multiple tracing providers.

## Why do we care?

Because there is no standardized way to propagate a trace, each tracing vendor has to create their own method of propagating tracing information. Typically this is in the form of a custom HTTP header. If an application request which is traced by one system flows through a system which is not traced using the same tracing platform, the headers may not be correctly propagated and the trace may be broken.

## Goals

- To provide a way in which heterogeneous architectures with multiple tracing platforms to operate in concert with each other
- To provide a method for application developers to trace requests without invading users' privacy

## Non-goals

TODO: add non-goals

## Concepts

### Distributed Trace
A distributed trace is the directed acyclic graph which represents a single operation, request, or user action, where each node in the graph is a span.
 
### Span
A span is a single operation within a distributed trace. For example, if a trace represents a user clicking on "Checkout," the request from the client to the server would be a single span, each request made by that server to other services would be a span, each database request would be a span, and so on.

### Trace Flags
Trace flags communicate information about the trace to remote tracing systems. Currently, the only information transmitted with the trace is whether or not a particular request is "sampled," or captured by a tracing system.

## Headers

### Traceparent

The traceparent header is a request header which contains a trace id, a parent span id, and trace flags. A tracing system transmits the trace ID, span ID, and the ID of each span's parent span, if it exists, with each span to a tracing backend. The tracing backend uses this information to construct the directed acyclic graph which represents the trace. In a system with a single tracing platform, it is the only header necessary to complete a distributed trace.

### Tracestate

The tracestate header is a request header which allows a tracing platform to transmit platform-specific information, even through systems which are traced using other tracing platforms. Each tracing platform uses a single key, and the value is treated as an opaque token by other tracing platforms. Each tracing platform is required to forward tracestate keys and values.

A typical use case might be: a request from host A to host B causes host B to make a request to host C. Host A and C are traced with one tracing platform, but host B is traced with a different platform. If host B is an active participant in the trace, the request from host B to host C will have a parent span ID which represents a span that is not reported to the tracing backend used by host A and C. To avoid this issue, host A could transmit the last known span ID in the tracestate, which is propagated unmodified to host C. Host C will notice that the span ID last seen by it's tracing platform is different than the parent span id in the traceparent header. It can use the last known span ID as its parent ID, and not break the trace.

TODO: sequence diagram

TODO: make tracestate example more concise if possible

### Unnamed Response Header (currently tracechild)

The tracechild header is a response header which contains the trace ID and trace flags used by the called service. This information may be used to link the trace in a third party called service to a first party trace, allow a proxy to delegate tracing decisions to the proxied service, or enable tail based sampling where a sampling decision is deferred until a request is completed and all information pertinent to the sampling decision is known.

TODO: proposed parent ID?

## Open Questions
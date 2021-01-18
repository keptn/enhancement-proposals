# Introduce OpenTelemetry pre-instrumentation to Keptn

Added OpenTelemetry pre-instrumentation to Keptn’s [helm-service](https://github.com/keptn/keptn/tree/master/helm-service) ensuring insights into e.g., response times and errors. 

## Motivation

Similar to Keptn, [OpenTelemetry](https://opentelemetry.io/) is an open-source CNCF project formed from the merger of the OpenCensus and OpenTracing projects. It provides a collection of tools, APIs, and SDKs for capturing metrics, distributed traces, and logs from applications. 
OpenTelemetry is expected to become the industry standard for:
*	pre-instrumenting libraries and frameworks to add out-of-the-box and vendor-neutral observability
*	enriching local monitoring data with custom instrumentation
While OpenTelemetry does not provide backend or analytics capabilities, it provides several integration points for observability platforms to ingest the collected data.

* Which use-cases does this KEP enable?
This KEP proposes to team up with the trending CNCF project OpenTelemetry by aiming to enrich Keptn’s [helm-service](https://github.com/keptn/keptn/tree/master/helm-service) with OpenTelemetry pre-instrumentation. The helm-service is used as first candidate to get pre-instrumentation since occasionally Keptn delivery use-cases fail due to missing insights into this particular service. Such pre-instrumentation shall ensure observability insights into e.g., response times and errors.

* Which value would this KEP create?
This KEP would add currently missing observability insights into the helm-service used for deploying services to a Kubernetes cluster and releasing them to user traffic.
Besides, it would extend the OpenTelemetry instrumentation for metrics already in place for the Keptn [statistics-service](https://github.com/keptn-sandbox/statistics-service).

* How can I, as a Keptn user, make use of the instrumentation
The value of OpenTelemetry pre-instrumentation is the following:
•	As a developer, adopting OpenTelemetry instrumentation allows to gain more insights into services or to add domain specific knowledge
•	OpenTelemetry data can be sent to an observability backend of choice for analysis to understand how apps are performing
•	As an SRE, OpenTelemetry instrumentation can be ingested into an observability backend of choice and used to ensure SLIs/SLOs are met for critical services

## Explanation
```
	// Start TCP server
    listener, _ := net.Listen("tcp", ":1234")
	conn, _ := listener.Accept()
	defer conn.Close()
	conn.Write([]byte("Hello from TCP server"))
	// Create custom Span that represent accepted TCP connection
	_, connSpan := tracer.Start(context.Background(), "Incoming TCP connection")
	defer connSpan.End()
	// Set client IP address as Span attribute
	connSpan.SetAttributes(label.KeyValue{
		Key:   "Client address",
		Value: label.StringValue(conn.LocalAddr().String()),
	})
```

The example above adds relevant custom metadata, i.e., client IP address to a low-level TCP connection. This shows how span attributes can be used to provide specific metadata related to, e.g., the helm-service. Such domain-specific metadata can be useful to quickly identify issues related to deploying services and mitigate them.

## Internal details

From a technical perspective, how do you propose accomplishing the proposal? 

In particular, please explain:

* How would the change impact and interact with existing functionality?
* Likely error modes and how to handle them
* Corner cases and how to handle them

While you do not need to prescribe a particular implementation - a KEP should be about **behavior**, not the implementation - it may be useful to provide at least one suggestion as to how the proposal *could* be implemented. This helps reassure reviewers that implementation is at least possible, and often helps them thinking more deeply about trade-offs, alternatives, etc.

## Trade-offs and mitigations

* Introduces a dependency to (probably unstable) OpenTelemetry APIs. The OpenTelemetry community is working hard towards providing release candidates for APIs (please see https://medium.com/opentelemetry/tracing-specification-release-candidate-ga-p-eec434d220f2 for all the details) 
* Ensure RCs are available for referenced APIs

## Breaking changes

Could this proposal cause any breaking changes (e.g., in the spec or within any workflow)?

## Prior art and alternatives

* What are some prior and/or alternative approaches? E.g., is there a corresponding feature in OpenTracing or OpenCensus? 
* What are some ideas that you have rejected?

## Open questions

Does OpenTelemetry pre-instrumentation for Keptn require to have trace context support available?

## Future possibilities

Of course, OpenTelemetry pre-instrumentation of other Keptn services could follow.

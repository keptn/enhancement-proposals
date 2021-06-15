# Keptn 2021 Roadmap

This is an incomplete list of work we hope to tackle in 2021. The *Roadmap* lists the main building blocks and derived KEPs. The *Timeline* when it is planned to work on the KEPs and how they are mapped to Keptn Releases.

## Roadmap

* **Shipyard v0.2**: *Clear separation of control-plane and execution-plane* & *Multi-cluster support*
  * [KEP 06](https://github.com/keptn/enhancement-proposals/pull/6) - The next generation of Shipyard
  * [KEP 37](https://github.com/keptn/enhancement-proposals/pull/37) - Closed-loop remediation with Shipyard v0.2
  * [KEP 39](https://github.com/keptn/enhancement-proposals/pull/37) - Advanced Sequence Handling in Control-plane
* **Keptn Hardening**
  * Secret endpoint: [#2829](https://github.com/keptn/keptn/discussions/2829)
  * Run containers as non-root user: [#3764](https://github.com/keptn/keptn/pull/3764)
* **Keptn Uniform support**: *Seamless integration of DevOps tooling into the sequence orchestration by Keptn.*
  * [KEP 42](https://github.com/keptn/enhancement-proposals/issues/42) - Basic Uniform support in Keptn and Visualization in Bridge (*Mockup*)
  * [KEP 45](https://github.com/keptn/enhancement-proposals/pull/45) - Troubleshooting support for Keptn-services (aka. Integrations)
  * [KEP 46](https://github.com/keptn/enhancement-proposals/pull/46) - Creating/Deleting Secrets for Integrations using Bridge
  * [KEP ++]() - Subscription management for Integrations
  * [KEP ##]() - Seamless integration of Webhook-based services
* **Zero-downtime Upgrades & High Availability**: *Running Keptn at scale without downtime for end-users.*
  * [KEP 48](https://github.com/keptn/enhancement-proposals/pull/48) - Zero-downtime Upgrades & High Availability
* **Alignment with [Continous Delivery Foundation (CDF) eventing standard](https://github.com/cdfoundation/sig-events)**: *Establish an industry-wide eventing standard jointly with the CDF.* 
  * [KEP **]() / [PR](https://github.com/cdfoundation/sig-events/pull/55) - Proof-of-Concept: Connect Tekton and Keptn by CDF events
* **Security: Access Control**: *Allow fine-grained access control for interacting with Keptn.*
  * Role-based Access Control - Read/Execution Access
  * Keptn API Token management
* **Execution-Plane support**: *Central component (e.g., Operator) for installing and managing execution-planes.*
  * Operator for Keptn-services (Integrations) on Execution Plane
* **Keptn Eco-system Expansion**
  * SDK for Integration Development
* (future) **Multi-tenancy**: *Reduction of resource footprint by multi-tenant capabilities.* 
  * Stepwise introduction of multi-tenancy capabilities

## Timeline / Release plan

| Building block        	| Start   	| End    	| finished KEP         	| Keptn Release     |
|-----------------------	|---------	|--------	|---------------------	|-----------------	|
| Shipyard v0.2                                       | 11/2020 	| 3/2021 	  | [KEP 06](https://github.com/keptn/enhancement-proposals/pull/6)             | [0.8.0](https://github.com/keptn/keptn/releases/tag/0.8.0) 	|
| Uniform Support                               | 3/2021  	| 3/2021 	  | [KEP 42](https://github.com/keptn/enhancement-proposals/pull/42) (UI part)  | [0.8.1](https://github.com/keptn/keptn/releases/tag/0.8.1) 	|
| *Keptn Hardening*  	                                | 3/2021  	| 4/2021 	  | -                    	                                                      | [0.8.2](https://github.com/keptn/keptn/releases/tag/0.8.2) 	|
| Shipyard v0.2                                       | 4/2021  	| 5/2021 	  | [KEP 37](https://github.com/keptn/enhancement-proposals/pull/37)            | [0.8.3](https://github.com/keptn/keptn/releases/tag/0.8.3) 	|
| Uniform Support                               | 5/2021  	| 6/2021 	  | [KEP 45](https://github.com/keptn/enhancement-proposals/pull/45) / [KEP 46](https://github.com/keptn/enhancement-proposals/pull/46) 	| 0.8.4 (in progress) |
| - Shipyard v2 <br> - Zero-downtime Upgrades & HA    | 6/2021  	| 7/2021 	  | [KEP 39](https://github.com/keptn/enhancement-proposals/pull/39)            | 0.8.5 (or 0.9.0)                         |
| - Uniform Support <br> - Zero-downtime Upgrades & HA  | 7/2021  	| 8/2021 	  | [KEP 42](https://github.com/keptn/enhancement-proposals/pull/42) (Core part), [KEP ++]()            | tbd                           |
| - Uniform Support <br> - Zero-downtime Upgrades & HA  | 8/2021  	| 9/2021 	  | [KEP ##]() 	                                                        | tbd                           |
| - Zero-downtime Upgrades & HA <br> - Security: AC           | 9/2021  	| 10/2021 	| [KEP 48](https://github.com/keptn/enhancement-proposals/pull/48) / x 	        | tbd                           |
| Keptn Execution Plane                               | 10/2021  	| 11/2021   | x                     | tbd                           |
| Keptn Eco-system Expansion                          | 11/2021  	| 11/2021   | x 	                  | tbd                           |
| Multi-tenancy Support                               | 11/2021  	| xx/2022   | x 	                  | tbd                           |

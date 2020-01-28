# Enriched Service Level Indicators

Enrich Service Level Indicator (SLI) with a set of values and a deep link to the data source.

## Motivation

In the current approach of implementing an SLI-provider*, the provider has to send out a Keptn CloudEvent of type [sh.keptn.internal.event.get-sli.done](https://github.com/keptn/spec/blob/master/cloudevents.md#get-sli-done) after querying the data from its data source. In the payload of this event, an array of indicators and their value is provided, e.g.:

```yaml
    "indicatorValues": [
      {
        "metric":"request_latency_p95",
        "value": 1.1620000000000001,
        "success": true
      },
      {
        "metric":"error_rate",
        "value": 0,
        "success": true
      }
    ],
```

The limitation is now that describing an SLI by one metric is too restricted. Most likely, more data points for an indicator are available that would provide an even better understanding of the indicator and support the visualization of the SLI. Additionally, a link for drilling into the tool (that is queried by the SLI provider) is useful to get a look at the actual (real) data set.

(* An SLI-provider is a vendor-specific service that queries data from a data source (e.g, from a monitoring or testing solution) using an SLI configuration.)

## Explanation

Instead of limiting an indicator to a single value, this KEP proposes to enrich the event payload of [sh.keptn.internal.event.get-sli.done](https://github.com/keptn/spec/blob/master/cloudevents.md#get-sli-done) with a set of values and a deep link for each indicator, as shown below:

```yaml
    "indicatorValues": [
      {
        "metric":"request_latency_p95",
        "values": [1.162, 1.651, 1.162, 1.653, 1.192, 1.191],
        "origin":"https://xyz.com/#smgd;sci=SERVICE-18E3F3E463387767;tab=RT;servicefilter=0%1E10%11SERVICE_METHOD_GROUP-175CF39880103983;gf=all;gtf=l_24_HOURS;timeframe=custom1580161200000to1580168400000"
        "success": true
      },
      {
        "metric":"error_rate",
        "values": [0, 1, 0, 0, 0, 0],
        "origin":"https://xyz.com/#smgd;sci=SERVICE-18E3F3E463387767;tab=RT;servicefilter=0%1E10%11SERVICE_METHOD_GROUP-175CF39880103983;gf=all;gtf=l_24_HOURS;timeframe=custom1580161200000to1580168400000"
        "success": true
      }
    ],
```

## Internal details

* *How the change would impact and interact with existing functionality?*: The lighthouse-service, which is responsible for processing SLIs, has to be capable of dealing with the set of values. Besides, the Keptn's bridge can be enhanced by showing the deep link next to the visualization of an SLI. 

## Trade-offs and mitigations

*What are some drawbacks? What are some ways that they might be mitigated?*: A drawback is the amount of data stored by Keptn. The internal data store of Keptn should not grow due to data that is not needed. There should be an agreement on how many values for an indicator are provided (see open question).

## Breaking changes

There is no breaking change since this KEP adds two properties to an event, but does not remove or rename a property.

## Open questions

Sending all data that is available for an indicator does not make sense since it is then a duplicate to the original data set. The origin of the data has to be the monitoring/testing tool that provides the data for the SLI-provider. It should be clarified, how many values make sense.

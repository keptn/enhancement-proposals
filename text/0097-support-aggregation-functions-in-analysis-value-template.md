# KEP 97 Support Aggregation functions in AnalysisValueTemplate

## Motivation

Currently, only the `KeptnMetrics` CRD supports the use of aggregation functions
(see [here](https://github.com/keptn/lifecycle-toolkit/blob/5fac158a7ffed67f7502fe03683138d717ea1acd/metrics-operator/api/v1beta1/keptnmetric_types.go#L76)).
Supporting the same aggregation functions might also be useful for `AnalysisValueTemplates` as well, 
to better support queries that return multiple data points of a time series. 
This way, the provider returning the results for the AnalysisValueTemplate can check whether
there are multiple data points in the result for the query and apply the aggregation function in that case.

## Proposed Design

The proposal is to extend the specification of the `AnalysisValueTemplates` with an `spec.aggregation` field,
similar to what is available in the `KeptnMetric` (see [specification](https://keptn.sh/latest/docs/reference/api-reference/metrics/v1beta1/#rangespec)).
Using this field, the metrics operator can inspect the result for an analysis value returned by a metrics provider,
and, if the aggregation function has been defined, use this function to aggregate the returned result if this is
an array of multiple values, as opposed to a single value.

## Benefits

- Unified way of defining aggregation functions for query results containing multiple time series data points
- More flexibility in terms of which queries can be used for each provider

## Open Questions

One argument that might speak against this feature is that usually monitoring provider
APIs support aggregation functions on their end already,
such as e.g. [DQL](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language/functions#aggregation-functions),
so it might be sufficient to adapt queries for AnalysisValueTemplates to make use of the monitoring provider's aggregation capabilities.

## References

See the following Github discussion for more context: https://github.com/keptn/lifecycle-toolkit/issues/2650#issuecomment-1906038916
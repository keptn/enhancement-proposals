# Change an SLO to an InfoSLO when SLI provides not data

Turn an SLO into an info SLO - i.e., an SLO without criteria - when the SLI does not provide data.   

## Motivation

* An SLI does not always provide metric values resutling in a faling SLO since data is expected. If an SRE knows the reason why the SLI is not providing data and this situation can be accpeted, he/she wants to ignore the SLO if no data is available. In other words, an SLO should be configurable to not check the criteria if no data is available.  

## Explanation

To configure an SLO to not check the criteria, the property `ignoreIfEmtpy` with value `true/false` is introduced: 

```
objectives:
- sli: response_time_p95
  ignoreIfEmtpy: true
  pass:
  - criteria:
    - "<=+10%"
    - "<600"
  warning:
  - criteria:
    - "<=800"
  weight: 2
  key_sli: true
```

* By default and if not specified, the property `ignoreIfEmtpy` is set to `false`.
* If `ignoreIfEmtpy` is set to `true` and the SLI, e.g., *response_time_p95*, does not return data, the pass and warning criteria are not checked (i.e., they are ignored). This results in an SLI that has just an informative character but plays no active part in the quality gate evaluation. 
* This is a spec change of the SLO configuration: https://github.com/keptn/spec/blob/master/service_level_objective.md

## Trade-offs and mitigations



## Breaking changes

- *no breaking change*: This introduces a spec change of the SLO which adds an additional property without breaking existing SLO configurations. 

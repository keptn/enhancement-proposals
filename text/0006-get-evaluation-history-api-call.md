# Provide an Evaluation Result History API Call

In order to build custom UI visualizations of the SLI/SLO based Evaluations to e.g: integrate with other CD Tools (Jenkins, Azure DevOps, Bitbucket ...) we need a Keptn API that delivers the same JSON as the Keptn Bridge shows underneath the Heatmap visualization

## Motivation

API-First Development:
The Keptns bridge is doing a great job in visualizing historical evaluation results - but - it is doing it by fetching data directly from the MongoDB. We should build this logic into an official API that the bridge is using and that others can use as well, e.g: users that want to visualize the data in a similar way in other tools such as Jenkins, Azure DevOps ...

## Details

Here some ideas on how that API Call could look like!

Examples:
Default: Last 20 runs
```
/project/sockshop/stage/dev/service/carts/evaluations
```

Last 10 runs that passed
```
/project/sockshop/stage/dev/service/carts/evaluations?lastRuns=10&status=pass
```

Runs between two timeslots
```
/project/sockshop/stage/dev/service/carts/evaluations?from=123123123&to=12312312
```

Single result from a specific Keptn Context
```
/project/sockshop/stage/dev/service/carts/evaluations?context=1231231231123
```

Last 20 Runs that have this specific label on it, e.g: last runs for a specific jenkins pipeline
```
/project/sockshop/stage/dev/service/carts/evaluations?label=jenkinspipelinesockshop
```

The result could be the same as it is currently in the keptns bridge when expanding the JSON of the Heatmap. It contains the full historical data:
```
{
    "data": {
      "evaluationHistory": [
      {
        "contenttype": "application/json",
        "data": {
          "deploymentstrategy": "direct",
          "evaluationdetails": {
            "indicatorResults": [
              {
                "score": 1,
                "status": "pass",
                "targets": [
            ....
              }
          }
        }
    }
}
```
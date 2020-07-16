# KEP0017 Declarative Configuration of Keptn

To make configurations more reproducable and storable in Git Repositories (e.g. GitOps approaches), we should introduce a declarative way to configure keptn.

## Motivation
* Sometimes, many commands have to be executed when configuring keptn
* It would be valuable if the keptn configuration could be checked in in a git reposory
* Duplicating projects by copying code-blocks would make some things easier

## Which use-cases does this KEP enable?
* Keptn could be configured via a single configuration file
* Configurations could be shared easily
* Configurations would be more reproducable 

## Explanation

As we know it from kubectl, among configuring things imperatively (e.g. keptn create service) it has some additional pros when making things configurable declaratively (via a config file or similar). A simple use case for that would be the preparation of demo environments, where the instructor only has to prepare a "keptnfile" which can be distributed to the attendees to prepare an environment.

## Internal details
To accomplish this, a first step would be the definition for such configurations. From my perspective, there would be two variants which would be feasible:

### A: Concourse-like
```
shipyards:
  - name: onboarding-carts
    stages:
      - name: "dev"
        deployment_strategy: "direct"
        test_strategy: "functional"
      - name: "staging"
        deployment_strategy: "blue_green_service"
        test_strategy: "performance"
      - name: "production"
        deployment_strategy: "blue_green_service"
        remediation_strategy: "automated"

slis:
  - name: sli-config-prom
    spec_version: '1.0'
    indicators:
      response_time_p50: histogram_quantile(0.5, sum by(le) (rate(http_response_time_milliseconds_bucket{handler="ItemsController.addToCart",job="$SERVICE-$PROJECT-$STAGE"}[$DURATION_SECONDS])))
      response_time_p90: histogram_quantile(0.9, sum by(le) (rate(http_response_time_milliseconds_bucket{handler="ItemsController.addToCart",job="$SERVICE-$PROJECT-$STAGE"}[$DURATION_SECONDS])))
      response_time_p95: histogram_quantile(0.95, sum by(le) (rate(http_response_time_milliseconds_bucket{handler="ItemsController.addToCart",job="$SERVICE-$PROJECT-$STAGE"}[$DURATION_SECONDS])))

slos:
  - name: slo-config-prom
    spec_version: "0.1.1"
    comparison:
      aggregate_function: "avg"
      compare_with: "single_result"
      include_result_with_score: "pass"
      number_of_comparison_results: 1
    filter:
    objectives:
      - sli: "response_time_p95"
        key_sli: false
        pass:             # pass if (relative change <= 10% AND absolute value is < 600ms)
          - criteria:
              - "<=+10%"  # relative values require a prefixed sign (plus or minus)
              - "<600"    # absolute values only require a logical operator
        warning:          # if the response time is below 800ms, the result should be a warning
          - criteria:
              - "<=800"
        weight: 1
    total_score:
      pass: "90%"
      warning: "75%"

projects:
  - name: sockshop
    shipyard: onboarding-carts
    services:
      - name: carts
        monitoring: prometheus
        stages:
          - name: hardening
            sli: sli-config-prom
            slo: slo-config-prom
          - name: production
            sli: sli-config-prom
            slo: slo-config-prom
  - name: bockshop
    shipyard: onboarding-carts
    services:
      - name: carts-x
        monitoring: prometheus
        stages:
          - name: hardening
            sli: sli-config-prom
            slo: slo-config-prom
          - name: production
            sli: sli-config-prom
            slo: slo-config-prom
```
### B: Kubernetes-like
```
apiVersion: keptn.sh/v1alpha1
kind: KeptnShipyard
metadata:
  name: onboarding-carts
spec:
  stages:
    - name: "dev"
      deployment_strategy: "direct"
      test_strategy: "functional"
    - name: "staging"
      deployment_strategy: "blue_green_service"
      test_strategy: "performance"
    - name: "production"
      deployment_strategy: "blue_green_service"
      remediation_strategy: "automated"

---
apiVersion: keptn.sh/v1alpha1
kind: KeptnSLI
metadata:
  name: sli-config-prom
spec:
  spec_version: '1.0'
  indicators:
    response_time_p50: histogram_quantile(0.5, sum by(le) (rate(http_response_time_milliseconds_bucket{handler="ItemsController.addToCart",job="$SERVICE-$PROJECT-$STAGE"}[$DURATION_SECONDS])))
    response_time_p90: histogram_quantile(0.9, sum by(le) (rate(http_response_time_milliseconds_bucket{handler="ItemsController.addToCart",job="$SERVICE-$PROJECT-$STAGE"}[$DURATION_SECONDS])))
    response_time_p95: histogram_quantile(0.95, sum by(le) (rate(http_response_time_milliseconds_bucket{handler="ItemsController.addToCart",job="$SERVICE-$PROJECT-$STAGE"}[$DURATION_SECONDS])))

---
apiVersion: keptn.sh/v1alpha1
kind: KeptnSLO
metadata:
  name: slo-config-prom
spec:
  spec_version: "0.1.1"
  comparison:
    aggregate_function: "avg"
    compare_with: "single_result"
    include_result_with_score: "pass"
    number_of_comparison_results: 1
  filter:
  objectives:
    - sli: "response_time_p95"
      key_sli: false
      pass:             # pass if (relative change <= 10% AND absolute value is < 600ms)
        - criteria:
            - "<=+10%"  # relative values require a prefixed sign (plus or minus)
            - "<600"    # absolute values only require a logical operator
      warning:          # if the response time is below 800ms, the result should be a warning
        - criteria:
            - "<=800"
      weight: 1
  total_score:
    pass: "90%"
    warning: "75%"

---
apiVersion: keptn.sh/v1alpha1
kind: KeptnService
metadata:
  name: carts
spec:
  monitoring: prometheus
  stages:
  - name: hardening
    sli: sli-config-prom
    slo: slo-config-prom
  - name: production
    sli: sli-config-prom
    slo: slo-config-prom

---
apiVersion: keptn.sh/v1alpha1
kind: KeptnProject
metadata:
  name: sockshop
spec:
  shipyard: onboarding-carts
  services:
    - name: carts
```

Afterwards, this configurations should be parsable through the CLI and therefore a user could apply them by typing `keptn apply -f <filename`.

In any way, we should be able to update configurations by re-applying such configuration files, but we could also consider making them deletable. Especially in the second variant, we should ensure that depending objects already exist in keptn, before applying the "new" ones. In the first variant this could be simply checked by parsing the configuration file.

In particular, please explain:

* How would the change impact and interact with existing functionality?
* Likely error modes and how to handle them
* Corner cases and how to handle them

## Breaking changes
Some present object types could be subject to change (e.g. by adding name fields).

## Open questions

What are some questions that you know aren't resolved yet by the KEP? 
* Data Structure
* How to deal with updates, deletions

## Future possibilities
* When using configuration variant b, the implementation of keptn objects as CRDs (with all its drawbacks)
* Enabling GitOps approaches

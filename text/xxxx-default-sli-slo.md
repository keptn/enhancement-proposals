# Add default sli and slo files with Keptn create and keptn onboard service

Automatically add default SLI.yaml and SLO.yaml when onboarding or creating a new service in Keptn

## Motivation

When starting with Keptn SLIs & SLOs are a bit like "magic". By default there is no visible SLI and therefore it is hard to understand what is happening behind the scenes and it is also hard to get started with a custom SLI because there is no good template to start from.
The motivation therefore is to make it easier for users to understand and extend SLIs & SLOs by automatically adding a default sli & slo when onboarding a new service

## Explanation

Whenever you create or onboard a service to Keptn, Keptn will automatically add a default set of SLIs and SLOs for your service. See this as "SLO driven development" where Keptn wants to you think about good SLOs from the start in order to bring your service into the right quality state.
If you want to modify your SLIs, e.g: adding new indicators, simply edit the SLI.yaml in your git repo.
If you want to modify your SLO, e.g: changing conditions or adding new SLIs, simply edit the SLO.yaml in your git repo.

## Internal details

I think the keptn API that is responsible for creating or onboarding a new service should automatically add a default SLI.yaml and SLO.yaml to all stages.
Technically this should be implemented by sending a message to configured SLI providers to return a list of default SLIs. In the future we could even extend this with additional concepts, e.g: send the technology type of the service (Java, php, node ...) and depending on the tech type the SLI provider can return different defaults. Another option would be to keep the templates in a public keptn git repo. Everytime a new services is onboarded/created Keptn could reach out to that Git repo and fetch the best matching template for the technology type of the service.

In particular, please explain:

The change wouldnt impact current functionality. It just extends the default behavior and makes understanding SLIs and SLOs easier as they become visible to the end user!
The output of the Keptn CLI should be clear that these resources have been added. If an upstream git was specified a link can even be provided as part of the output to point the user to it.
I also think we should make the SLI and SLO more prominent in the Bridge when navigating through services and stages. We could provide a link or even an "Edit SLI/SLO" in the bridge!

## Trade-offs and mitigations

Don't think there are trade-offs as this simply enhances the understanding of SLIs & SLOs - where they are defined and what the format looksl ike

## Breaking changes

No breaking changes

## Prior art and alternatives

Alternatives are more documentation!

## Open questions

Shall we provide a capability like this for existing projects that have been created in the past, e.g: via keptn update service that will then add the SLI/SLO to the repo

## Future possibilities

As mentioned above - we could think of technology specific template SLIs, SLOs that we could pull from a public repo. 
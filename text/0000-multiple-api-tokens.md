# Generate Multiple API Tokens

The ability to generate multiple Keptn API tokens

## Motivation

* All Keptn integrations use one API token.
* If the API token is regenerated, all Keptn services stop working and need to be redeployed with the new token

## Explanation

Today, all Keptn integrations (aka services) utilise a single API token. This is insecure as it is a single point of failure and a shared secret.

If this single API token is leaked or for some other reason needs to be regenerated, all Keptn services stop working and need to be redeployed with the new token.

## Internal details

Implementation of this KEP would see the ability to generate multiple API tokens on demand, so (for example) a user can generate tokens for each service individually.

A user can also void an API token at will knowing it will only affect a smaller subset of components.

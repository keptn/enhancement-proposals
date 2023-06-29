# KEP 90: Functions - Runtime Adaptions

**State: IMPLEMENTED**

## Motivation
Keptn Functions provide an easy way to add functionality to the delivery process. At the moment, the runtime is implemented in a very simple way, it is a simple container which executes deno functions. If the function succeeeds, everything is fine, if it fails, the container exits with an error code. This behavior has some drawbacks:

* It might be intended that a function fails some times (e.g. if a dependency is not available). At the moment, this leads to Crash Looping Pods, which is not the desired behavior as this might be misleading for observability solutions.
* At the moment the function runtime does not provide any information about the execution of the function. This makes it hard to debug functions, as the user has to look into the logs of the function container.
* Currently, only Deno functions are supported which might be a limitation for some users and use cases.

Given the above, the goal of this KEP is to provide a more robust runtime for functions, which provides more information about the execution of the function and which is more flexible in terms of supported languages.

## Current Design
Currently, the function runtime is implemented as a simple container which executes the function and passes over some environment variables, as user-provided data, secrets and context information. The current implementation can be found [here](https://github.com/keptn/lifecycle-toolkit/tree/main/functions-runtime).

## Assumptions / Definitions
* When executing a script, the user has to pass over information about the language. The default will be `deno`
* The Function Runtime should support deno, bash and python scripts
* It is possible to return a JSON object from the function
* The function runtime stays in a single container
* It is possible to pass over a timeout, retry interval and number of retries to the function runtime

## Proposed Design
To determine the language of the function, the respective language will be passed over as environment variable (RUNTIME). The function runtime will wrap the function execution and is responsible for the following tasks:
* Execute the function using the respective language runtime (deno, python, bash)
* If the function fails, the function runtime will retry the execution of the function (number of retries, retry interval, timeout)
* Provide information about the execution of the function (e.g. execution time, number of retries, error message, etc.) to the lifecycle controller

Therefore, the following fields have to be added in the Function Struct on the lifecycle toolkit:
```
Retries: int
RetryInterval: int
Timeout: int
Runtime: string (default: deno, possible values: deno, bash, python)
```

The Retries, RetryInterval and Timeout specify the behavior of the function runtime in case the function fails. As long as the fail count and the timeout is not reached, the function will be retried without restarting the container. When the timeout or fail count is reached, the container will exit with an error code and the Task will be marked as failed. After each execution, status information can be written to a status file, which is mounted into the container. This file is a JSON object which could contain the following fields:

```
{
  "status": "succeeded" | "failed",
  "message": "string",
  "timestamp": "string"
}
``` 

This information will be passed over to the KeptnTask CRD (to be investigated), to provide more information about the execution of the function.

For the bash runtime, following packages are proposed at the moment: `curl, jq, git`, for python: `requests, json, git, yaml` might be a good starting point.

## Benefits
* Additional languages will be supported, which means more flexibility for creating functions
* More information about the task execution
* Friendlier behavior in case of function failures

## Open Questions
* How to pass over the status information to the KeptnTask CRD?
* Should the different runtimes be implemented in different multiple images or should they be implemented in one container image?
* How far should the function runtime for bash and python be hardened?
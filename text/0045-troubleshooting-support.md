# Troubleshooting support for Keptn-service (aka. Integration)

**Success Criteria:** Support for troubleshooting errors in Keptn-services (aka. Integrations)

## Motivation

* *Pain*: A Keptn user does not get any information about errors that happened in a Keptn-service (aka. Integration)
* *Target*: Keptn provides basic support to list errors that occurred in a Keptn-service.
* *Driver*: Improve the experience for working with Keptn-services. 

## Use Case (1): 

#### (1) As a user, I would like to see all errors that occurred in a Keptn-service in the Keptn Bridge.

*Type of Errors*: In a Keptn-service, two types of errors can occur: (1) task-related and (2) not-task-related once. 
* *Task-related:* When executing a specific task, an error occurs. This already results in a `xyz.finished` event with status `errored`. 
* *Non-task-related:* For example, the Keptn-service can not start, or an API endpoint can`t be reached. This functionality is not related to the execution of a task. 

--> Regardless of the context in which the error happened, I want to get informed about the error in the Keptn Bridge.

*User flow in Bridge*:

* Open the Uniform of a project
* All Keptn-services (Integrations) that are connected to the Control Plane and subscribed to this project are listed. 
* An indicator in the column *Status* tells me that a service has problems: see `errored` in red;
* When clicking on the service, the last 10 errors that happened in this Keptn-service are listed.
    * An error is displayed by a *red icon*, the *timestamp*, and the *error message*.
    * By default, just the first line of the *error message* is shown. When clicking on it, I get the entire message.  
* A `show older Errors` loads the next 10 errors. 

![image](https://user-images.githubusercontent.com/729071/117654267-c9ded700-b195-11eb-8650-f4957a6ec149.png)

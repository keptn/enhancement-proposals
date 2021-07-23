# Creating/Deleting Secrets for Integrations using Bridge

## Motivation

* *Pain*: Creating a secret for a Keptn-service requires the Keptn CLI or API. This can be a problem; especially in restricted environments where the download of the CLI from GitHub is restricted.
* *Target*: UI-based approach to create/delete a secret for a Keptn-service. 
* *Driver*: Improve the experience in working with Keptn-services; reduce context-switches between UI & CLI.

## Use Case (1) - (2): 

:warning: This feature is supported only for Keptn-services (integrations) that run on the control plane. Consequently, secrets for external Keptn-services are not managed.

#### (1) As a user, I would like to create a secret for a Keptn-service (aka Integration) using the Keptn Bridge.

*User flow in Bridge:*
* Open the Uniform of a project and select the **Secrets** tab. 
* Add a secret using the provided form: 
  * Specify the `Name` and at least one `key:value` pair
  * Add multiple `key:value` pairs by clicking on the + symbol
  * By clicking the *Add Secret* button, the secret is created and listed above. 
  * After successfully creating the secret, the form is empty. 
*Details:*
  * By default, the value field hides the entered value. 
  * If the value should be revealed, use the "eye" icon.

![image](https://user-images.githubusercontent.com/729071/117795505-8ac08c80-b24e-11eb-83a0-13feb48fb7fc.png)

* Switch the tab and go to **Services**. 
* This view shows all all Keptn-services (Integrations) that are connected to the Control Plane and subscribed to this project
* Select the Keptn-service that should get access to the created secret
* In the *Secrets* section, give the service permissions to access the secret

![image](https://user-images.githubusercontent.com/729071/117795997-f7d42200-b24e-11eb-836f-d797dc3937e6.png)


#### (2) As a user, I would like to delete a secret of a Keptn-service (aka Integration) using the Keptn Bridge.

*User flow in Bridge:*
* Select the **Secrets** tab and click the "X" to delete the particular secret.
* If the secret is in use by a Keptn-service, show a notification to inform the user and provide an option to `Cancel` and to `Delete Secret`:

![image](https://user-images.githubusercontent.com/729071/117797319-28688b80-b250-11eb-9dd9-cde249351add.png)


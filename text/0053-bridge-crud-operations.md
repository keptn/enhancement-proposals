# Manage Keptn entities using Bridge

## Motivation

* *Pain*: While a Keptn user can use the CLI/API for performing all operations, the UI has to provide the CRUD operations for project and service. 

* *Target*: Implementation of the main functionalities to create/update/delete a project/service.

* *Driver*:  Make it easier for Keptn users to get started and guide the user through a "getting-started" journey.

## User Cases: 

### As a user, create a new project with a Shipyard containing default sequences.

* Starting point: Display the `Create Project` button and *ready for an automation journey* message (if no project has been created yet) on the project screen:

![image](https://user-images.githubusercontent.com/729071/122953886-4234dc80-d37f-11eb-85e6-338cf6230b4d.png)

* Forward user to the settings page of a project that behaves as :
  * Instead of the project name the term `new project` is shown on the top
  * Menu on the left is disabled 
  * Mandatory fields are: project name and a Shipyard with at least one stage
  * If the mandatory fields are provided, the `Create Project` button gets activated

![image](https://user-images.githubusercontent.com/729071/122954492-c5563280-d37f-11eb-9f54-f4ec471a6823.png)

* After creating a project:
  * A notification is displayed on the top
  * The project name is displayed next to `keptn / `
  * The delete this project feature is available, and 
  * The menu on the right is activated. 

![image](https://user-images.githubusercontent.com/729071/122954834-1108dc00-d380-11eb-8132-8213d862cf44.png)

---

### As a user, I would like to delete a project.

* After creating a project, the user can delete a project via the settings page:
  * A dialog box is asking for confirmation and shows an error in case deleting the project did not work (see second box): 

![image](https://user-images.githubusercontent.com/729071/122955491-7230af80-d380-11eb-810a-7a5b6d4b714c.png)

* After deleting the project, the user is forwarded to the project overview where a notification informs about the successful deletion: 

![image](https://user-images.githubusercontent.com/729071/122955938-dbb0be00-d380-11eb-8690-c9c9e4bb89d6.png)

---

### As a user, I would like to create a service.

There will be the following three entry points for creating a service:

1. Forward the user to the "settings page" of a service by following the link here:
  ![image](https://user-images.githubusercontent.com/729071/122956570-68f41280-d381-11eb-8ff5-02575b4d6be3.png)
1.  Forward the user to the "settings page" of a service by clicking on the `Create Service` button on the service screen: 
  ![image](https://user-images.githubusercontent.com/729071/122956822-a2c51900-d381-11eb-84d2-8675f821258d.png)
1. On the settings page under "Services"

The view to create a service is located in the project settings. The user gets re-directed here.

On the view, there is a form field, a drag and drop element and a button "Create service"
* The form field "Service name" is mandatory.
* **HOW TO DEAL WITH FILES?** (see below) 
* When the button is clicked, the service is created via API.
* If there is an error, an error message is displayed on the same view.
* In the case of successfully creating the service, the service gets added to the navigation. The navigation points get preselected and the service name is shown as the headline in the view.

---

### As a user, I would like to delete a service. 

* By going to the settings page and selecting a service, the user can delete it:

![image](https://user-images.githubusercontent.com/729071/122958276-000d9a00-d383-11eb-881d-cc8f13b95b92.png)

* When selecting a service, a `Danger zone` section is provided that allows deleting the service
* A confirmation dialog asks the user to enter the name of the service
* After entering the name, the button (I understand ... ) is active
* If an error occurred, the error is shown in the confirmation dialog:

![image](https://user-images.githubusercontent.com/729071/122252930-022bb080-cecc-11eb-97a0-cfddccc4703d.png)

* After successfully deleting a service, the user stays at the settings page
* A notification informs the user about a successful deletion: 

![image](https://user-images.githubusercontent.com/729071/126137667-317fa252-1ad3-4fe5-a04a-bbbd469aa884.png)

* Finally, the next service in the list gets selected (in this case: carts-db)

---

### As a user, I would like to trigger a sequence.

* There are two starting points for triggering a sequence in the Bridge:
  * Environment screen - here I have the overview of the whole project
  * Sequence screen - context-wise I would expect to be able to trigger a sequence also here

* When the user clicks on the button `Trigger a new sequence`
  * Environment screen: Opens the form
  * Sequence screen: Redirects to environment screen with an open form

![image](https://user-images.githubusercontent.com/729071/152109673-9923d3dd-bb66-4054-b9ad-742d028081f2.png)


# Version awareness in Keptn

**Success Criteria**: **

## Short abstract

After implementing [KEP-81](https://github.com/keptn/enhancement-proposals/pull/81) and [KEP-70](https://github.com/keptn/enhancement-proposals/pull/70), this enhancement proposal of introducing service/config versioning in Keptn makes the switch to the folder-based approach sound and applicable. Please note that this has already been discussed in a broader scope in [KEP-58](https://github.com/keptn/enhancement-proposals/pull/58).

```
------------          ------------          ------------
|          |          |          |          |          |
|  KEP-81  |--------->|  KEP-70  |--------->| >KEP-82< |
|          |          |          |          |          |
------------          ------------          ------------
``` 

### TL;DR:
* Versioning of configuration is needed to allow a user to trigger a delivery without knowing the Git commit Id. 
* Uploading config is a bulk operation that takes a ZIP or URL to a `./keptn` folder. 

## Why

### Target audience / Pain points / Related discussions

**Target audience:**

* DevOps engineer, who has to set up a delivery process for and end-to-end delivery.

**Pain:** 

* *How does a Keptn user know which version (!= Git commit id) to deploy?*

  * *Why is this actually a problem?* Because it could be the case that a user has no access to the Git repo storing the configuration and hence the Git commit is unknown.

* Currently, Keptn derives the version information from the `image:tag`. Relying on this tag is a too opinionated approach because delivery must not always be for an image. 

**Related discussions/issues:**

* [Introduce version as additional meta-data for a service](https://github.com/keptn/keptn/issues/7420)

## What

Please consider the derived drawing that is be explained below: 

![Versioning in Keptn drawio](https://user-images.githubusercontent.com/729071/163342376-a82680f2-5bdb-4473-94c0-fd2a7ee1029a.png)

* Starting in the top left corner, we see a CI pipeline that is building, testing artifacts, and finally updating configuration to deploy. While in Keptn 0.14 this configuration is uploaded using `keptn add-resource` or manually committed to the upstream repo, uploading it via a `/config` endpoint that knows a `service`, `version` and `config` parameter would make it much easier. 
* The `config` parameter supports two formats:
  1) A ZIP archive following a folder structure that stores the config files as explained below.
  2) A URL that points to a `./keptn` folder with the same structure. This `./keptn` folder could be part of another repository, e.g., a source code repository that is used for service development.
* The `resource-service` processes the API request and stores the configuration in Git. Basically, it extracts or downloads the config and pushes it to the Git repository. The folder structure could look as follows - but need to be clarified: 
    ```
    ./base
    ./services
      |- carts
      |- payment
    ./stages
      |- dev
      |- hardening
      |- production
    ```
* Besides, the `resource-service` tags the commit using the service name and version: `svc-version`. It is important to have the service name as part of the tag since the repo is managing multiple services that eventually have the same version at some point in time. However, with the service name it becomes a unique key. 
* (*Future - just an idea for now*): Another variant of the `resource-service` could use an S3 bucket as the backend with an implementation of the versioning approach for that kind of storage. This could be based on the file name. 
* Now, let's assume a Keptn user (or CI) wants to trigger a deployment for carts in `v0.2.0`. **How does the person know that this version is stored with the Git commit `#85a3f2c`**? A very challenging task or actually not possible when the repo is not accessible. The only meta-data that is known to the person is the version; i.e., `v0.2.0`. Consequently, the only information that can be provided to Keptn at this point of time. 
* When executing the delivery sequence, this version number `v0.2.0` could be mapped to the Git commit id `#85a3f2c`. At the end, this is a technical detail. 

### What it is?

* Introducing `version` property in keptn/spec
* (Configuration can be versioned)
* Bulk upload of configuration with version attached
* Allow triggering a sequence containing the version
  ```
  {
    "data": {
      "project": "sockshop",
      "service": "carts",
      "stage": "dev",
      "version": "0.13.3",
     [ ... ]
    }
  }
  ```
* Bridge uses `version` with fall back to `image:tag` for displaying version information
* Bridge uses `version` with fall back to `buildId` for displaying version information (https://keptn.sh/docs/0.14.x/bridge/#services-view) 

#### ⭐ Use Case: As a user, I can perform a bulk upload of configuration which has a version attached

![image](https://user-images.githubusercontent.com/729071/164679362-3fa1c844-d553-43ee-a3cf-158eddab691d.png)

#### ⭐ Use Case: As a user, I can trigger a sequence with a reference to the version

This trigger: 

![image](https://user-images.githubusercontent.com/729071/164676565-dd4d433f-0abb-4503-a804-314a17c68e90.png)

will become the event: 

```
{
  "data": {
    "project": "sockshop",
    "service": "carts",
    "stage": "dev",
    "version": "0.13.3",
   [ ... ]
  }
}
```

#### ⭐ Improvement: Bridge uses `version` continuously through the different screens

![image](https://user-images.githubusercontent.com/729071/164676029-5ed0477e-3050-46c9-aee4-e7f03b3ce999.png)

### What it is not?

## Open questions

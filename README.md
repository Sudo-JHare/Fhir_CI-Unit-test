#   Fhir_CI-Unit-test

This repository contains a GitHub Actions workflow designed to automate the deployment of a HAPI FHIR server, load Implementation Guides (IGs) and test data, and build/push a Docker image.

##   Workflow Overview

The workflow is defined in `.github/workflows/main.yml` and is triggered by:

* `push` events on the `main` branch.
* `pull_request` events on the `main` branch.
* `workflow_dispatch` events (manual triggering).

##   Configuration File

The workflow now reads configuration values from a `config.properties` file located in the root of the repository. This file is used to set environment variables for the workflow.

##   Workflow Steps

The workflow consists of the following key steps:

1.  **Checkout code:** Checks out the repository code.
2.  **Set up JDK:** Sets up the specified Java Development Kit (JDK) version. 
3.  **Load Configuration:** Reads the `config.properties` file and sets environment variables.
4.  **Build and Push Docker Image:**
    * This step is executed only on `push` events to the `main` branch.
    * It builds a Docker image from the `Dockerfile` in the repository.
    * The image is tagged with the Git SHA and pushed to Docker Hub.
    * Optionally, it also tags and pushes the image with the `latest` tag.
5.  **Start HAPI FHIR Server:**
    * Pulls the latest HAPI FHIR Docker image. 
    * Runs the HAPI FHIR server in a Docker container. 
6.  **Wait for HAPI FHIR Server to start:** Waits until the HAPI FHIR server is ready to accept connections. 
7.  **Setup .NET SDK:** Sets up the .NET SDK. 
8.  **Install UploadFIG:** Installs the `UploadFIG` tool globally. 
9.  **Debug Environment Variables:** Prints the values of key environment variables for debugging. 
10. **Upload IG using UploadFIG:** Uploads the Implementation Guide to the HAPI FHIR server using the `UploadFIG` tool. 
11. **Clone Test Data:** Clones the repository containing the test data. 
12. **Load Test Data:**
    * Downloads the latest Linux executable of the TestDataClient from the releases in the `au-fhir-test-data-utils` repository. 
    * Loads the test data into the HAPI FHIR server using the `TestDataClient` tool. 
13. **Verify Test Data Loaded:** Verifies that the test data was loaded successfully (e.g., by querying the server for a specific resource type). 

##   Environment Variables

The workflow uses the following environment variables, which are now loaded from the `config.properties` file:

* `HAPI_VERSION`: The version of the HAPI FHIR server. 
* `IG_REPO`: The GitHub repository containing the Implementation Guide. 
* `TEST_DATA_REPO`: The GitHub repository containing the test data. 
* `UPLOADFIG_CONFIG`: Path to the UploadFIG configuration file. 
* `IG_PACKAGE_ID`: The IG Package ID. 
* `IG_PACKAGE_VERSION`: The IG Package Version. 
* `HAPI_SERVER`: The URL of the HAPI FHIR server. 
* `TERM_SERVER`: The URL of the terminology server. 

##   Docker Image

The workflow builds and pushes a Docker image to Docker Hub (or your specified registry) when code is pushed to the `main` branch. The image is tagged with the Git SHA of the commit.

##   Usage

To use this workflow:

1.  Ensure you have a `Dockerfile` in your repository.
2.  Create a `config.properties` file in the root of your repository and populate it with the necessary configuration values.
3.  Commit the `config.properties` file to your repository.

##   License

This project is licensed under the CC0 1.0 Universal license.

To view a copy of this license, visit:

[https://creativecommons.org/publicdomain/zero/1.0/](https://creativecommons.org/publicdomain/zero/1.0/)

#   Fhir_CI-Unit-test

This repository contains a GitHub Actions workflow designed to automate the deployment of a HAPI FHIR server, load Implementation Guides (IGs) and test data, and build/push a Docker image.

##   Workflow Overview

The workflow is defined in `.github/workflows/main.yml` and is triggered by:

* `push` events on the `main` branch.
* `pull_request` events on the `main` branch.
* `workflow_dispatch` events (manual triggering).

##   Workflow Inputs

The workflow accepts the following inputs, which can be provided when manually triggering it:

* `ig_repo`: The GitHub repository containing the Implementation Guide (IG) files (default: `hl7au/au-fhir-ig`).
* `test_data_repo`: The GitHub repository containing the test data (default: `hl7au/au-fhir-test-data`).
* `uploadfig_config`: Path to the UploadFIG configuration file (default: `uploadfig.json`).
* `ig_package_id`: The IG Package ID (default: `hl7.fhir.au.core`).
* `ig_package_version`: The IG Package Version (default: `1.1.0-preview`).
* `hapi_server`: The URL of the HAPI FHIR server (default: `http://localhost:8080/fhir`).
* `terminology_server`: The URL of the terminology server (default: `https://api.healthterminologies.gov.au/integration/R4/fhir`).
* `docker_username`: Docker Hub username (for pushing the Docker image).
* `docker_password`: Docker Hub password or token.

##   Workflow Steps

The workflow consists of the following key steps:

1.  **Checkout code:** Checks out the repository code.
2.  **Set up JDK:** Sets up the specified Java Development Kit (JDK) version.
3.  **Build and Push Docker Image:**
    * This step is executed only on `push` events to the `main` branch.
    * It builds a Docker image from the `Dockerfile` in the repository.
    * The image is tagged with the Git SHA and pushed to Docker Hub.
    * Optionally, it also tags and pushes the image with the `latest` tag.
4.  **Start HAPI FHIR Server:**
    * Pulls the latest HAPI FHIR Docker image.
    * Runs the HAPI FHIR server in a Docker container.
5.  **Wait for HAPI FHIR Server to start:** Waits until the HAPI FHIR server is ready to accept connections.
6.  **Setup .NET SDK:** Sets up the .NET SDK.
7.  **Install UploadFIG:** Installs the `UploadFIG` tool globally.
8.  **Debug Environment Variables:** Prints the values of key environment variables for debugging.
9.  **Upload IG using UploadFIG:** Uploads the Implementation Guide to the HAPI FHIR server using the `UploadFIG` tool.
10. **Clone Test Data:** Clones the repository containing the test data.
11. **Load Test Data:**
    * Downloads the latest Linux executable of the TestDataClient from the releases in the `au-fhir-test-data-utils` repository.
    * Loads the test data into the HAPI FHIR server using the `TestDataClient` tool.
12. **Verify Test Data Loaded:** Verifies that the test data was loaded successfully (e.g., by querying the server for a specific resource type).

##   Docker Image Build and Push Details

The `Build and Push Docker Image` step adds Docker image building and pushing functionality to the workflow. Here's a breakdown:

* **Inputs for Docker Credentials:**
    * Two new inputs (`docker_username` and `docker_password`) are added to the `workflow_dispatch` section. This allows providing Docker registry credentials when manually triggering the workflow.
* **Environment Variables for Docker Credentials:**
    * Corresponding environment variables (`DOCKER_USERNAME` and `DOCKER_PASSWORD`) are added to the `env` section of the `deploy_and_load` job. These variables retrieve the values from the workflow inputs.
* **Build and Push Docker Image Step:**
    * A new step named `Build and Push Docker Image` is included.
    * `if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}`: This condition ensures the step runs only when code is pushed to the `main` branch (you can adjust this).
    * `echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin`: This command logs in to your Docker registry. **Important:** While generally safe in GitHub Actions, avoid exposing passwords in logs whenever possible. Consider using GitHub Secrets for sensitive data if your registry supports it.
    * `docker build -t your-username/your-image-name:${{ github.sha }} .`: This command builds the Docker image.
        * `your-username/your-image-name`: Replace this with your Docker Hub username and image name.
        * `${{ github.sha }}`: The Git commit SHA is used as a unique image tag for versioning.
        * `.`: Specifies the current directory (where the `Dockerfile` is) as the build context.
    * `docker push your-username/your-image-name:${{ github.sha }}`: Pushes the image to the registry using the Git SHA tag.
    * **Optional: Tagging `latest` (Use with Caution):**
        * `docker tag ... :${{ github.sha }} your-username/your-image-name:latest`: Creates a `latest` tag.
        * `docker push ... :latest`: Pushes the image with the `latest` tag.
        * **Caution:** The `latest` tag is mutable and can cause issues in production. Immutable tags (like Git SHAs) are strongly recommended for production deployments.
* **Before You Use This:**
    * **Replace Placeholders:** Replace `your-username/your-image-name` with your actual Docker Hub details.
    * **Dockerfile:** Verify your `Dockerfile` is correctly configured.
    * **Registry Credentials:** Provide Docker Hub username/password or adjust the `docker login` command for other registries.
    * **Security:** Use GitHub Secrets to manage Docker credentials securely.
    * **Branching Strategy:** Adapt the `if` condition to your branching workflow.

This enhanced workflow automates building and pushing Docker images on `main` branch pushes. Adapt it carefully for your specific needs and security practices.

##   Environment Variables

The workflow uses the following environment variables:

* `HAPI_VERSION`: The version of the HAPI FHIR server.
* `IG_REPO`: The GitHub repository containing the Implementation Guide.
* `TEST_DATA_REPO`: The GitHub repository containing the test data.
* `UPLOADFIG_CONFIG`: Path to the UploadFIG configuration file.
* `IG_PACKAGE_ID`: The IG Package ID.
* `IG_PACKAGE_VERSION`: The IG Package Version.
* `HAPI_SERVER`: The URL of the HAPI FHIR server.
* `TERM_SERVER`: The URL of the terminology server.
* `DOCKER_USERNAME`: Docker Hub username.
* `DOCKER_PASSWORD`: Docker Hub password or token.

##   Usage

To use this workflow:

1.  Ensure you have a `Dockerfile` in your repository.
2.  Configure the workflow inputs (either by modifying `.github/workflows/main.yml` or by providing them when manually triggering the workflow).
3.  Provide your Docker registry credentials as inputs when triggering the workflow.

##   License

This project is licensed under the CC0 1.0 Universal license.

To view a copy of this license, visit:

[https://creativecommons.org/publicdomain/zero/1.0/](https://creativecommons.org/publicdomain/zero/1.0/)

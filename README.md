# Deploying a Docker Container to Google Cloud Run

## Prerequisites

* A Google Cloud account (you can sign up for a free trial)
* A basic understanding of Docker concepts

## Project Goal

Build a Docker container from a Dockerfile, push it to Google Cloud Artifact Registry, and deploy it to Cloud Run.

## 1. Getting Started with Google Cloud Shell

* Open Google Cloud Shell in your browser: [https://console.cloud.google.com](https://console.cloud.google.com) and activate it by clicking the "Activate Cloud Shell" button at the top.
* Google Cloud Shell: This is a web-based terminal environment that lets you run commands directly on Google Cloud resources. It eliminates the need to install any software locally.
* **Important:**  All commands below are meant to be run in the Cloud Shell terminal.

## 2. Set Up Environment

* Project ID: Every Google Cloud project has a unique identifier. You'll need to replace your-project-id with your actual project ID. You can find your project ID in the Google Cloud Console (look for the project name at the top).

* Region: Google Cloud data centers are spread across the globe in different regions. Choose a region closest to your users for optimal performance. We'll use us-west1 as an example throughout this guide, but you can select a different region if needed.

* The commands set these values as environment variables, making them easily accessible in later steps.

```bash
# Replace with your Google Cloud project's ID
export PROJECT_ID=your-project-id

# Choose your desired region (stick with one throughout) 
export REGION=us-west1
```

## 3. Authenticate

* gcloud auth login: This command prompts you to log in to your Google Cloud account. This is necessary to grant you permission to use Google Cloud resources.

* gcloud auth configure-docker: This command connects your local Docker installation to Google Cloud. It allows you to push your Docker images to Artifact Registry and deploy them to Cloud Run.

```
gcloud auth login 
# Follow instructions to log in to your Google Cloud account

gcloud auth configure-docker 
# Authorizes Docker to interact with Google Cloud
```

## 4. Enable Necessary APIs

* APIs (Application Programming Interfaces):  These are tools that let different services communicate with each other. In this case, we need to enable two APIs:
	* Artifact Registry API: This grants permission to interact with Artifact Registry, Google Cloud's service for storing Docker images.
	* Cloud Run API: This grants permission to interact with Cloud Run, Google Cloud's service for deploying containerized applications.
* The gcloud services enable commands activate these APIs for your project.

```
gcloud services enable artifactregistry.googleapis.com 
gcloud services enable run.googleapis.com
```

## 5. Create a Docker Repository

* Docker Repository:  This is a virtual storage location specifically designed for Docker images.  Think of it as a library for your containerized applications.
* The gcloud artifacts repositories create command creates a new Docker repository named my-docker-repo within your project. You can customize this name if desired.
* The additional flags specify that this is a Docker repository, set the location to your chosen region ($REGION), and associate it with your project ($PROJECT_ID).

```
gcloud artifacts repositories create my-docker-repo \
    --repository-format=docker \
    --location=$REGION \ 
    --project=$PROJECT_ID
```

## 6. Build the Docker Image

* Dockerfile: This is a text file that contains instructions for building a Docker image. It specifies the operating system, dependencies, and application code to be included in the final image.
* docker build -t my-image .: This command tells Docker to build an image using the Dockerfile in your current directory and tag it with the name my-image.

```
docker build -t my-image .  # Assumes Dockerfile is in your current directory
```

## 7. Tag the Image

* Image Tag: A tag is a way to label a specific version of a Docker image. Here, we're creating a tag that includes the region ($REGION) and the Artifact Registry path to your repository. This helps Cloud Run locate the image later.
* The docker tag command creates a new tag named $REGION-docker.pkg.dev/$PROJECT_ID/my-docker-repo/my-image:latest that points to the same underlying image as my-image.

```
docker tag my-image $REGION-docker.pkg.dev/$PROJECT_ID/my-docker-repo/my-image:latest
```

## 8. Push the Image to Artifact Registry

* Push: This refers to the process of transferring the Docker image to your Artifact Registry repository.
* The docker push command sends your my-image:latest image to the Artifact Registry location specified in the tag.

```
docker push $REGION-docker.pkg.dev/$PROJECT_ID/my-docker-repo/my-image:latest
```

## 9. Deploy to Cloud Run

* Cloud Run: This is a Google Cloud service that lets you deploy containerized applications without managing servers.
* The gcloud run deploy command instructs Cloud Run to deploy your containerized application based on the image stored in Artifact Registry.

```
gcloud run deploy --image $REGION-docker.pkg.dev/$PROJECT_ID/my-docker-repo/my-image:latest \
    --project=$PROJECT_ID \
    --region=$REGION \
    --platform managed \
    --allow-unauthenticated
```

You've deployed your container. You'll see the URL to access it in the output.
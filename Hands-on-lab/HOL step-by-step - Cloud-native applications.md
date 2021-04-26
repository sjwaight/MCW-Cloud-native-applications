![Microsoft Cloud Workshops][logo]

<div class="MCWHeader1">
Cloud-native applications
</div>

<div class="MCWHeader2">
Hands-on lab step-by-step
</div>

<div class="MCWHeader3">
January 2021
</div>

Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2021 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

**Contents**

<!-- TOC -->
- [Cloud-native applications - Developer edition hands-on lab step-by-step](#cloud-native-applications---developer-edition-hands-on-lab-step-by-step)
  - [Abstract and learning objectives](#abstract-and-learning-objectives)
  - [Overview](#overview)
  - [Solution architecture](#solution-architecture)
  - [Requirements](#requirements)
  - [Exercise 1: Create and build Docker images](#exercise-1-create-and-build-docker-images)
    - [Task 1: Create a Dockerfile](#task-1-create-a-dockerfile)
    - [Task 2: Create Docker images](#task-2-create-docker-images)
    - [Task 3: Run a containerized application](#task-3-run-a-containerized-application)
    - [Task 4: Setup environment variables](#task-4-setup-environment-variables)
    - [Task 5: Setup Continous Integration Pipeline to Push Images](#task-5-setup-continous-integration-pipeline-to-build-and-push-images)
  - [Exercise 2: Migrate MongoDB to Cosmos DB using Azure Database Migration Service](#exercise-2-migrate-mongodb-to-cosmos-db-using-azure-database-migration-service)
    - [Task 1: Enable Microsoft.DataMigration resource provider](#task-1-enable-microsoftdatamigration-resource-provider)
    - [Task 2: Provision Azure Database Migration Service](#task-2-provision-azure-database-migration-service)
    - [Task 3: Migrate data to Azure Cosmos DB](#task-3-migrate-data-to-azure-cosmos-db)
  - [Exercise 3: Deploy the solution to Azure Kubernetes Service](#exercise-3-deploy-the-solution-to-azure-kubernetes-service)
    - [Task 1: Deploy a service using the Azure Portal](#task-1-deploy-a-service-using-the-azure-portal)
    - [Task 2: Deploy a service using kubectl](#task-2-deploy-a-service-using-kubectl)
    - [Task 3: Deploy a service using a Helm chart](#task-3-deploy-a-service-using-a-helm-chart)
    - [Task 4: Configure Continuous Delivery to the Kubernetes Cluster](#task-4-configure-continuous-delivery-to-the-kubernetes-cluster)
    - [Task 5: Review Azure Monitor for Containers](#task-5s-review-azure-monitor-for-containers)
  - [Exercise 4: Scale the application and test High Availability](#exercise-4-scale-the-application-and-test-high-availability)
    - [Task 1: Increase service instances from the Azure Portal](#task-1-increase-service-instances-from-the-azure-portal)
    - [Task 2: Resolve failed provisioning of replicas](#task-2-resolve-failed-provisioning-of-replicas)
    - [Task 3: Restart containers and test High Availability](#task-3-restart-containers-and-test-high-availability)
    - [Task 4: Configure Cosmos DB Autoscale](#task-4-configure-cosmos-db-autoscale)
    - [Task 5: Test Cosmos DB Autoscale](#task-5-test-cosmos-db-autoscale)
  - [Exercise 5: Working with services and routing application traffic](#exercise-5-working-with-services-and-routing-application-traffic)
    - [Task 1: Scale a service without port constraints](#task-1-scale-a-service-without-port-constraints)
    - [Task 2: Adjust CPU constraints to improve scale](#task-2-adjust-cpu-constraints-to-improve-scale)
    - [Task 3: Perform a rolling update](#task-3-perform-a-rolling-update)
    - [Task 4: Configure Kubernetes Ingress](#task-4-configure-kubernetes-ingress)
    - [Task 5: Multi-region Load Balancing with Traffic Manager](#task-5-multi-region-load-balancing-with-traffic-manager)
  - [After the hands-on lab](#after-the-hands-on-lab)
<!-- /TOC -->

# Cloud-native applications - Developer edition hands-on lab step-by-step

## Abstract and learning objectives

This hands-on lab is designed to guide you through the process of building and deploying Docker images to the Kubernetes platform hosted on Azure Kubernetes Services (AKS), in addition to learning how to work with dynamic service discovery, service scale-out, and high-availability.

At the end of this lab, you will be better able to build and deploy containerized applications to Azure Kubernetes Service and perform common DevOps procedures.

## Overview

Fabrikam Medical Conferences (FabMedical) provides conference website services tailored to the medical community. They are refactoring their application code, based on node.js, so that it can run as a Docker application, and want to implement a POC that will help them get familiar with the development process, lifecycle of deployment, and critical aspects of the hosting environment. They will be deploying their applications to Azure Kubernetes Service and want to learn how to deploy containers in a dynamically load-balanced manner, discover containers, and scale them on demand.

In this hands-on lab, you will assist with completing this POC with a subset of the application codebase. You will create a build agent based on Linux, and an Azure Kubernetes Service cluster for running deployed applications. You will be helping them to complete the Docker setup for their application, test locally, push to an image repository, deploy to the cluster, and test load-balancing and scale.

> **Important**: Most Azure resources require unique names. Throughout these steps, you will see the word "SUFFIX" as part of resource names. You should replace this with a unique handle (like your Microsoft Account email prefix) to ensure unique names for resources.

## Solution architecture

Below is a diagram of the solution architecture you will build in this lab. Please study this carefully, so you understand the whole of the solution as you are working on the various components.

The solution will use Azure Kubernetes Service (AKS), which means that the container cluster topology is provisioned according to the number of requested nodes. The proposed containers deployed to the cluster are illustrated below with Cosmos DB as a managed service:

![A diagram showing the solution, using Azure Kubernetes Service with a Cosmos DB back end.](media/solution-topology.png "Solution architecture diagram")

Each tenant will have the following containers:

- **Conference Web site**: The SPA application that will use configuration settings to handle custom styles for the tenant.

- **Admin Web site**: The SPA application that conference owners use to manage conference configuration details, manage attendee registrations, manage campaigns, and communicate with attendees.

- **Registration service**: The API that handles all registration activities creating new conference registrations with the appropriate package selections and associated cost.

- **Email service**: The API that handles email notifications to conference attendees during registration, or when the conference owners choose to engage the attendees through their admin site.

- **Config service**: The API that handles conference configuration settings such as dates, locations, pricing tables, early-bird specials, countdowns, and related.

- **Content service**: The API that handles content for the conference such as speakers, sessions, workshops, and sponsors.

## Requirements

1. Completed the steps as per ["Before the hands-on lab setup guide"](Before%20the%20HOL%20-%20Cloud-native%20applications.md) 

2. Local machine or a virtual machine configured with:

   - A browser, preferably the new Microsoft Edge or Chrome for consistency with the lab implementation tests.

3. You will install other tools throughout the exercises.

   > **Very important**: You should be typing all the commands as they appear in the guide. Do not try to copy and paste to your command windows or other documents when instructed to enter the information shown in this document, except where explicitly stated in this document. There can be issues with Copy and Paste that result in errors, execution of instructions, or creation of file content.

## Exercise 1: Create and build Docker images

**Duration**: 40 minutes

In this exercise, you will create a Dockerfile for the existing application components, build Docker images, and run containers to execute the application.

### Help references

|                                            |                                                                                           |
| ------------------------------------------ | :---------------------------------------------------------------------------------------: |
| **Description**                            | **Links**                                                                                 |
| Getting started with Docker                | https://docs.docker.com/get-started/                                                      |
| About registries, repositories, and images | https://docs.microsoft.com/azure/container-registry/container-registry-concepts           |
| Publishing Docker images                   | https://docs.github.com/free-pro-team@latest/actions/guides/publishing-docker-images   |

### Task 1: Create a Dockerfile

In this task, you will create a new Dockerfile that will be used to build a container image for the Content API application.

1. Open a new SSH connection to your lab Virtual Machine. 

   ```bash
   ssh -i .ssh/fabmedical adminfabmedical@HOST_IP_ADDRESS
   Enter passphrase for key '.ssh/fabmedical':
   ```

   > **Note:** 
   >   1. By default Azure VM IP addresses change if they are stopped. If you stopped your VM and didn't reserve the IP address or set a hostname then you should check the new IP address of the lab machine.
   >
   >  2. If you performed the optional setup task an wish to use Visual Studio Code Remote Development then [follow the documentation](https://code.visualstudio.com/docs/remote/ssh#_connect-to-a-remote-host) on how to connect. If you run into issues check the SSH config hasn't stripped the dot and path from the key file.
   >
   >  3. If you did not install Visual Studio Code then you will be working with `Vim` for a few editing exercises. Steps are included on how to work with files.

2. Navigate to the `content-api` folder. List the files in the folder with this command. The output should look like the screenshot below.

   ```bash
   cd ../content-api
   ll
   ```

   ![In this screenshot of the console window, ll has been typed and run at the command prompt. The files in the folder are listed in the window. At this time, we are unable to capture all of the information in the window. Future versions of this course should address this.](media/image55.png "List the files")

2. Create a new file named `Dockerfile` (uppercase D). The window should look as shown in the following screenshot.

   ```bash
   vi Dockerfile
   ```

   ![This is a screenshot of a new file named Dockerfile in the console window.](media/image56.png "Open new file in VIM")

   > **Note:** if you are using Visual Studio Code then you can select to open the remote folder and navigate as necessary. Select the **New file** button and enter **Dockerfile** to create the new file remotely. 

3. Vim users: type `i` on your keyboard to put Vim into insert mode. You will see the bottom of the window showing INSERT mode.

   ![INSERT appears at the bottom of the Dockerfile window.](media/image57.png "Insert mode")

4. Type the following into the file. These statements are described below.

   > **Note**: Type the following into the Vim editor, as you may have errors with copying and pasting. Visual Studio Code users can copy and paste.

   ```Dockerfile
   FROM node:alpine AS base
   RUN apk -U add curl
   WORKDIR /usr/src/app
   EXPOSE 3001

   FROM node:argon AS build
   WORKDIR /usr/src/app

   # Install app dependencies
   COPY package.json /usr/src/app/
   RUN npm install

   # Bundle app source
   COPY . /usr/src/app

   FROM base AS final
   WORKDIR /usr/src/app
   COPY --from=build /usr/src/app .
   CMD [ "npm", "start" ]
   ```

This Dockerfile describes a multi-stage build of the following:

   - The base stage includes environment setup which we expect to change very rarely, if at all.

     - Creates a new Docker image from the base image `node:alpine`. This base image has node.js on it and is optimized for small size.

     - Add `curl` to the base image to support Docker health checks.

     - Creates a directory on the image where the application files can be copied.

     - Exposes application TCP port `3001` to the container environment so that the application can be reached at port `3001`.

   - The build stage contains all the tools and intermediate files needed to create the application.

     - Creates a new Docker image from `node:argon`.

     - Creates a directory on the image where the application files can be copied.

     - Copies `package.json` to the working directory.

     - Runs `npm install` to initialize the node application environment.

     - Copies the source files for the application over to the image.

   - The final stage combines the base image with the build output from the build stage.

     - Sets the working directory to the application file location.

     - Copies the app files from the build stage.

     - Indicates the command to start the node application when the container is run.

5. Vim users: when you are finished typing, hit the Esc key and type `:wq` and hit the Enter key to save the changes and close the file. 

   ```bash
   <Esc>
   :wq
   <Enter>
   ```

6. List the contents of the folder again to verify that the new Dockerfile has been created.

   ```bash
   ll
   ```

   ![In this screenshot of the console window, ll has been typed and run at the command prompt. The Dockerfile file is highlighted at the top of list.](media/image58.png "Highlight the Dockerfile")

7. Vim users: verify the file contents to ensure it was saved as expected. Type the following command to see the output of the Dockerfile in the command window. If it doesn't appear correct you can delete using `rm Dockerfile` and try again.

   ```bash
   cat Dockerfile
   ```

### Task 2: Create Docker images

In this task, you will create local Docker images for the application --- one for the API application and another for the web application. Each image will be created via Docker commands that rely on a Dockerfile.

1. From SSH prompt connected to the lab VM, type the following command to view any Docker images on the VM.

   ```bash
   docker image ls
   ```

   > **Note:** if you are using Visual Studio Code you can open a remote Terminal by selecting Terminal > New Terminal. You can then run this and follow commands.

2. From the content-api folder containing the API application files and the new Dockerfile you created, type the following command to create a Docker image for the API application. This command does the following:

   - Executes the Docker build command to produce the image

   - Tags the resulting image with the name `content-api` (-t)

   - The final dot (`.`) indicates to use the Dockerfile in this current directory context. By default, this file is expected to have the name `Dockerfile` (case sensitive).

   ```bash
   docker image build -t content-api .
   ```

3. Once the image is successfully built, run the Docker images listing command again. You will see several new images: the node images and your container image.

   ```bash
   docker image ls
   ```

   Notice the untagged image. This is the build stage which contains all the intermediate files not needed in your final image.

   ![The node image (node) and your container image (content-api) are visible in this screenshot of the console window.](media/image59.png "List Docker images")

4. Commit and push the new Dockerfile before continuing.

   ```bash
   git add .
   git commit -m "Added Dockerfile"
   git push
   ```

   Enter credentials if prompted. In Visual Studio Code you can use the Git Extension to commit and push to GitHub.

   > **Note:** you may receive an email about a failed build on GitHub Actions. You can ignore this for now as we will address later.

5. Navigate to the content-web folder again and list the files. Note that this folder already has a Dockerfile.

   ```bash
   cd ../content-web
   ll
   ```

6. View the Dockerfile contents -- which are similar to the file you created previously in the API folder. Type the following command:

   ```bash
   cat Dockerfile
   ```

   > Notice that the `content-web` Dockerfile build stage includes additional tools for a front-end Angular application in addition to installing npm packages.

7. Type the following command to create a Docker image for the web application.

   ```bash
   docker image build -t content-web .
   ```
8. When complete, you will see eight images now exist when you run the Docker images command.

   ```bash
   docker image ls
   ```

### Task 3: Run a containerized application

The web application container will be calling endpoints exposed by the API application container and the API application container will be communicating with a containerized MongoDB instance.

1. On your lab VM create a Docker network named `fabmedical`:

   ```bash
   docker network create fabmedical
   ```

2. Run a MongoDB container instance for local testing. The image will be downloaded automatically if required.

   ```bash
   docker container run --name mongo --net fabmedical -p 27017:27017 -d mongo:4.0
   ```

3. Once completed you can test that the mongodb instance is running using the following commands

   ```bash
   mongo
   show dbs
   quit()
   ```

   ![Screenshot showing commands to access local MongoDB instance and check status.](media/Ex1-Task1.5.png "MongoDB test")

4. Initialize the MongoDB database with test content as follows.

   ```bash
   cd ../content-init
   npm install
   ```

   > **Note**: In some cases, the `root` user will be assigned ownership of your user's `.config` folder. If this happens, run the following command to return ownership to `adminfabmedical` and then try `npm install` again:

   ```bash
   sudo chown -R $USER:$(id -gn $USER) /home/adminfabmedical/.config
   ```

5. Initialize the database. Sample output is shown below.

   ```bash
   nodejs server.js
   ```

   ![Screenshot showing commands to populate MongoDB instance with data.](media/populate-mongo.png "Output from populating MongoDB")

6. Confirm database was populated.

   ```bash
   mongo
   show dbs
   use contentdb
   show collections
   db.speakers.find()
   db.sessions.find()
   quit()
   ```

7. Create and start the API application container with the following command. The command does the following:

   - Names the container `api` for later reference with Docker commands.

   - Instructs the Docker engine to use the `fabmedical` network.

   - Instructs the Docker engine to use port `3001` and map that to the internal container port `3001`.

   - Creates a container from the specified image, by its tag, such as `content-api`.

   ```bash
   docker container run --name api --net fabmedical -p 3001:3001 content-api
   ```

8. The `docker container run` command has failed because it is configured to connect to MongoDB using a localhost URL. However, now that content-api is isolated in a separate container, it cannot access MongoDB via localhost even when running on the same docker host. Instead, the API must use the bridge network to connect to MongoDB.

   ```bash
   > content-api@0.0.0 start
   > node ./server.js

   Listening on port 3001
   Could not connect to MongoDB!
   MongooseServerSelectionError: connect ECONNREFUSED 127.0.0.1:27017
   npm notice
   npm notice New patch version of npm available! 7.0.8 -> 7.0.13
   npm notice Changelog: <https://github.com/npm/cli/releases/tag/v7.0.13>
   npm notice Run `npm install -g npm@7.0.13` to update!
   npm notice
   npm ERR! code 255
   npm ERR! path /usr/src/app
   npm ERR! command failed
   npm ERR! command sh -c node ./server.js

   npm ERR! A complete log of this run can be found in:
   npm ERR!     /root/.npm/_logs/2020-11-23T03_04_12_948Z-debug.log
   ```

9. The content-api application allows an environment variable to configure the MongoDB connection string. Remove the existing container, and then instruct the docker engine to set the environment variable by adding the `-e` switch to the `docker container run` command. Also, use the `-d` switch to run the api as a daemon.

   ```bash
   docker container rm api
   docker container run --name api --net fabmedical -p 3001:3001 -e MONGODB_CONNECTION=mongodb://mongo:27017/contentdb -d content-api
   ```

10. Enter the command to show running containers. You will observe that the `api` container is in the list. Use the docker logs command to see that the API application has connected to MongoDB.

      ```bash
      docker container ls
      docker container logs api
      ```

   ![In this screenshot of the console window, docker container ls has been typed and run at the command prompt, and the "api" container is in the list with the following values for Container ID, Image, Command, Created, Status, Ports, and Names: 458d47f2aaf1, content-api, "docker-entrypoint.s...", 37 seconds ago, Up 36 seconds, 0.0.0.0:3001->3001/tcp, and api.](media/image61.png "List Docker containers")

11. Test the API by curling the URL. You will see JSON output containing speaker information.

      ```bash
      curl http://localhost:3001/speakers
      ```

12. Create and start the web application container with a similar `docker container run` command -- instruct the docker engine to use any port with the `-P` command.

      ```bash
      docker container run --name web --net fabmedical -P -d content-web
      ```

13. Enter the command to show running containers again, and you will observe that both the API and web containers are in the list. The web container shows a dynamically assigned port mapping to its internal container port `3000`.

      ```bash
      docker container ls
      ```

   ![In this screenshot of the console window, docker container ls has again been typed and run at the command prompt. 0.0.0.0:32768->3000/tcp is highlighted under Ports.](media/image62.png "List Docker containers")

14. Test the web application by fetching the URL with curl. For the port, use the dynamically assigned port, which you can find in the output from the previous command. You will see HTML output, as you did when testing previously.

      ```bash
      curl http://localhost:32768/speakers.html
      ```

      > **Note:** the port number (32768) may be different in your enviornment, so check output from Step 15.

### Task 4: Setup environment variables

In this task, you will configure the Web container so that it can use an environment variable for the URL to use to communicate with the API container. This is similar to the way the MongoDB connection string is provided to the API container.

1. On your lab VM stop and remove the web container using the following commands.

   ```bash
   docker container stop web
   docker container rm web
   ```

2. Validate that the web container is no longer running or present by using the `-a` flag as shown in this command. You will see that the `web` container is no longer listed.

   ```bash
   docker container ls -a
   ```

3. Open the content-web folder and review the `app.js` file.

   ```bash
   cd ../content-web
   cat app.js
   ```

4. Observe that the `contentApiUrl` variable can be set with an environment variable.

   ```javascript
   const contentApiUrl = process.env.CONTENT_API_URL || "http://localhost:3001";
   ```

5. Open the Dockerfile for editing using Vim and press the `i` key to go into edit mode. You can also edit it in Visual Studio Code remotely if you have it installed.

   ```bash
   vi Dockerfile
   ```

6. Locate the `EXPOSE` line shown below and add a line above it that sets the default value for the environment variable, as shown in the screenshot.

   ```Dockerfile
   ENV CONTENT_API_URL http://localhost:3001
   ```

   ![In this screenshot of Dockerfile, the CONTENT_API_URL code appears above the next Dockerfile line, which reads EXPOSE 3000.](media/hol-2019-10-01_19-37-35.png "Set ENV variable")

7. Press the Escape key and type `:wq` and then press the Enter key to save and close the file.

   ```text
   <Esc>
   :wq
   <Enter>
   ```

8. Rebuild the web application Docker image using the same command as you did previously.

   ```bash
   docker image build -t content-web .
   ```

9. Create and start the image passing the correct URI to the API container as an environment variable. This variable will address the API application using its container name over the Docker network you created. After running the container, check to see the container is running and note the dynamic port assignment for the next step.

   ```bash
   docker container run --name web --net fabmedical -P -d -e CONTENT_API_URL=http://api:3001 content-web
   docker container ls
   ```

10. Curl the speakers path again, using the port assigned to the web container. Again, you will see HTML returned, but because curl does not process javascript, you cannot determine if the web application is communicating with the api application. You must verify this connection in a browser.

    ```bash
    curl http://localhost:[PORT]/speakers.html
    ```

11. You will not be able to browse to the web application on the ephemeral port because the lab VM only exposes a limited port range. Now you will stop the web container and restart it using port `3000` to test in the browser. Type the following commands to stop the container, remove it, and run it again using explicit settings for the port.

    ```bash
    docker container stop web
    docker container rm web
    docker container run --name web --net fabmedical -p 3000:3000 -d -e CONTENT_API_URL=http://api:3001 content-web
    ```

12. Curl the speaker path again, using port `3000`. You will see the same HTML returned.

    ```bash
    curl http://localhost:3000/speakers.html
    ```

13. You can now use a web browser to navigate to the website and successfully view the application at port `3000`. Replace `[LAB_VM_IP]` with the **IP address** of the virtual machine (this is the same IP you have used for SSH).

    ```bash
    http://[LAB_VM_IP]:3000

    EXAMPLE: http://13.68.113.176:3000
    ```

   Navigate to the Speakers page and you will see the results.

   ![In this screenshot we see the resulting web application running on a Container on an Azure VM.](media/web-app.png "Speaker page on website.")


14. Commit your changes and push to the repository.

    ```bash
    git add .
    git commit -m "Setup Environment Variables"
    git push
    ```

    Enter credentials if prompted.

### Task 5: Setup Continous Integration Pipeline to Build and Push Images

To run containers in a remote environment, you will typically push images to a Container Registry, where you can store and distribute images. In our scenario each service (web and API) will have a repository in a Registry that can be pushed to and pulled from using standard Docker commands. We will use Azure Container Registry (ACR) which is a managed private Docker registry service based on Docker Registry v2.

In this task, you will use GitHub Actions to build your Docker images and pushes them to your ACR instance automatically.

1. In the [Azure Portal](https://portal.azure.com/), navigate to the **Azure Container Registry** that was created for you by the ARM template you executed in the "Before the hands-on lab" exercise.

2. Select **Access keys** under **Settings** on the left-hand menu.

   ![In this screenshot of the left-hand menu, Access keys is highlighted below Settings.](media/image64.png "Access keys")

3. The Access keys blade displays the Login server, username, and password that will be required for the next step. Keep this handy as you perform actions on the build VM.

   > **Note**: If the username and password do not appear, select _Enable_ on the _Admin_ user option.

4. In another browser tab open GitHub, and open your **Fabmedical** repository. Select the **Settings** tab.

2. From the left menu, select **Secrets**.

3. Select the **New repository secret** button.

    ![Settings link, Secrets link, and New secret button are highlighted.](media/2020-08-24-21-45-42.png "GitHub Repository secrets")

4. In the **New secret** form, enter the name `ACR_USERNAME` and for the value, paste in the Azure Container Registry **Username** that was copied previously. Select **Add secret**.

    ![New secret screen with values are entered.](media/2020-08-24-21-48-54.png "New secret screen")

5. Add another Secret, by entering the name `ACR_PASSWORD` and for the value, paste in the Azure Container Registry **Password** that was copied previously.

    ![Secrets screen with both the ACR_USERNAME and ACR_PASSWORD secrets created.](media/2020-08-24-21-51-24.png "Secrets screen")

6. On your lab VM, navigate to the main `Fabmedical/.github/workflows` directory which contains our existing GitHub Actions defintions.

7. We next need to  create the GitHub Action definition for our `content-web` container. If you are using Visual Studio Code you can simply create a new file, otherwise with Vim do the following:

    ```dotnetcli
    vi content-web.yml
    ```

   Add the following as the content. Be sure to replace the following placeholders:

   - replace `[SHORT_SUFFIX]` with your short suffix such as `SOL`.

   ```yml
   name: content-web

   # This workflow is triggered on push to the 'content-web' directory of the  master branch of the repository
   on:
      push:
         branches:
            - master
         paths:
            - 'content-web/**'

      # Configure workflow to also support triggering manually
      workflow_dispatch:

    # Environment variables are defined so that they can be used throughout the job definitions.
    env:
      imageRepository: 'content-web'
      resourceGroupName: 'fabmedical-[SHORT_SUFFIX]'
      containerRegistryName: 'fabmedical[SHORT_SUFFIX]'
      containerRegistry: 'fabmedical[SHORT_SUFFIX].azurecr.io'
      dockerfilePath: './content-web'
      tag: '${{ github.run_id  }}'

   # Jobs define the actions that take place when code is pushed to the master branch
   jobs:
      build-and-publish-docker-image:
        name: Build and Push Docker Image
        runs-on: ubuntu-latest
        steps:
        # Checkout the repo
        - uses: actions/checkout@master

        - name: Set up Docker Buildx
          uses: docker/setup-buildx-action@v1

        - name: Login to ACR
          uses: docker/login-action@v1
          with:
            registry: ${{ env.containerRegistry }}
            username: ${{ secrets.ACR_USERNAME }}
            password: ${{ secrets.ACR_PASSWORD }}

        - name: Build and push an image to container registry
          uses: docker/build-push-action@v2
          with:
            context: ${{ env.dockerfilePath  }}
            file: "${{ env.dockerfilePath }}/Dockerfile"
            pull: true
            push: true
            tags: |
              ${{ env.containerRegistry }}/${{ env.imageRepository }}:${{ env.tag }}
              ${{ env.containerRegistry }}/${{ env.imageRepository }}:latest
    ```

   > Note: you can [download this as a YAML file](lab-files/yaml-templates/ex1-task5-actions.yaml) and use the file as your template. Make sure to update [SHORT_SUFFIX].

10. Save the file and exit Vi by pressing `<Esc>` then `:wq`.

11. Save the pipeline YAML, then commit and push it to the Git repository:

    ```bash
    git add .
    git commit -m "Added content-web Action definition"
    git push
    ```

12. In GitHub, return to the **Fabmedical** repository screen, and select the **Actions** tab.

13. On the **Actions** page, select the **content-web** workflow.

14. On the **content-web** workflow, select **Run workflow** and manually trigger the workflow to execute.

    ![The content-web Action is shown with the Actions, content-web, and Run workflow links highlighted.](media/2020-08-25-15-38-06.png "content-web workflow")

15. After a second, the newly triggered workflow execution will display in the list. Select the new **content-web** execution to view its status.

16. Selecting the **Build and Push Docker Image** job of the workflow will display its execution status.

    ![Build and Push Docker Image job.](media/2020-08-25-15-42-11.png "Build and Push Docker Image job")

17. Next, setup the `content-api` workflow. This repository already includes `content-api.yml` located within the `.github/workflows` directory. Open the `.github/workflows/content-api.yml` file for editing.

18. Edit the `resourceGroupName` and `containerRegistry` environment values to replace `[SHORT_SUFFIX]` with your own three-letter suffix so that it matches your container registry's name and resource group.

    ![The screenshot shows the content-api.yml with the environment variables highlighted.](media/2020-08-25-15-59-56.png "content-api.yml environment variables highlighted")

19. Save the file, then navigate to the repositories in GitHub, select Actions, and then manually run the **content-api** workflow.

20. Next, setup the **content-init** workflow. Follow the same steps as the previous `content-api` workflow for the `content-init.yml` file, remembering to update the `[SHORT_SUFFIX]` value with your own three-letter suffix.

21. Commit and push the changes to the Git repository:

   ```bash
   git add .
   git commit -m "Updated workflow YAML"
   git push
   ```

## Exercise 2: Migrate MongoDB to Cosmos DB using Azure Database Migration Service

**Duration**: 20 minutes

At this point, you have the web and API applications containerized. The next step is to migrate the MongoDB data over to Azure Cosmos DB. This exercise will use the Azure Database Migration Service to migrate the data from the MongoDB database into Azure Cosmos DB.

> **Note:** in order to complete this Exercise the MongoDB container you ran on your lab VM in Exercise 1, Task 2 must still be running.

### Help references

|                                            |                                                                                           |
| ------------------------------------------ | :---------------------------------------------------------------------------------------: |
| **Description**                            | **Links**                                                                                 |
| What is Azure Database Migration Service?  | https://docs.microsoft.com/azure/dms/dms-overview                                         |
| Azure Cosmos DB's API for MongoDB          | https://docs.microsoft.com/azure/cosmos-db/mongodb-introduction |
| Tutorial: Migrate MongoDB to Azure Cosmos DB's API for MongoDB offline using DMS | https://docs.microsoft.com/azure/dms/tutorial-mongodb-cosmos-db |

### Task 1: Enable Microsoft.DataMigration resource provider

In this task, you will use Azure Database Migration Service within your Azure subscription by registering the `Microsoft.DataMigration` resource provider.

1. Open the Azure Cloud Shell.

2. Run the following Azure CLI command to register the `Microsoft.DataMigration` resource provider in your Azure subscription:

   ```sh
   az provider register --namespace Microsoft.DataMigration
   ```

### Task 2: Provision Azure Database Migration Service

In this task, you will deploy an instance of the Azure Database Migration Service that will be used to migrate the data from MongoDB to Cosmos DB.

1. Open the [Azure Portal Create Migration Service Blade](https://portal.azure.com/#create/Microsoft.AzureDMS).

2. On the **Basics** tab of the **Create Migration Service** pane, enter the following values:

    - Resource group: Select the Resource Group created with this lab.
    - Migration service name: Enter a name, such as `fabmedical[SUFFIX]`.
    - Location: Choose the Azure Region used for the Resource Group.
    - Service mode: leave **Azure** selected.

    ![The screenshot shows the Create Migration Service Basics tab with all values entered.](media/dms-create-basics.png "Create Migration Basics Tab")

3. Select **Next: Networking >>**.

6. On the **Networking** tab, select the **Virtual Network** within the `fabmedical-[SUFFIX]` resource group.

    ![The screenshot shows the Create Migration Service Networking tab with Virtual Network selected.](media/dms-create-networking.png "Create Migration Service Networking tab")

7. Select **Review + create**.

8. Select **Create** to create the Azure Database Migration Service instance.

The service may take 5 - 10 minutes to provision.

### Task 3: Migrate data to Azure Cosmos DB

In this task, you will create a **Migration project** within Azure Database Migration Service, and then migrate the data from MongoDB to Azure Cosmos DB.

1. In the Azure Portal, navigate to the **Azure Database Migration Service** that was previously provisioned.

2. On the Azure Database Migration Service blade, select **+ New Migration Project** on the **Overview** pane.

3. On the **New migration project** pane, enter the following values, then select **Create and run activity**:

    - Project name: `fabmedical`
    - Source server type: `MongoDB`
    - Target server type will be automatically set to: `CosmosDB (MongoDB API)`
    - Choose type of activity: `Offline data migration`

    ![The screenshot shows the New migration project pane with values entered.](media/dms-new-migration-project.png "New migration project pane")

    > **Note:** The **Offline data migration** activity type is selected since you will be performing a one-time migration from MongoDB to Cosmos DB. Also, the data in the database won't be updated during the migration. In a production scenario, you will want to choose the migration project activity type that best fits your solution requirements.

4. Select **Create and run activity**. Wait while the job is created and you will be redirected to the migration wizard.

5. On the **MongoDB to Azure Database for CosmosDB Offline Migration Wizard** pane, enter the following values for the **Select source** tab:

    - Mode: **Standard mode**
    - Source server name: Enter the Private IP Address of the lab VM running your MongoDB Containers.
    - Server port: `27017`
    - Require SSL: Unchecked

    > **Note:** Leave the **User Name** and **Password** blank as the MongoDB instance on the Build Agent VM for this lab does not have authentication turned on. The Azure Database Migration Service is connected to the same VNet as the lab VM, so it's able to communicate within the VNet directly to the VM without exposing the MongoDB service to the Internet. In production scenarios, you should always have authentication enabled on MongoDB.

    ![Select source tab with values selected for the MongoDB server.](media/dms-select-source.png "MongoDB to Azure Database for CosmosDB - Select source")

5. Select **Next: Select target >>**. This may take some time to complete as the Migration Service attempts to connect to the MongoDB instance on the lab VM.

6. On the **Select target** tab, select the following values:

    - Mode: **Select Cosmos DB target**

    - Subscription: Select the Azure subscription you're using for this lab.

    - Select Cosmos DB name: Select the `fabmedical-[SUFFIX]` Cosmos DB instance.

    ![The Select target tab with values selected.](media/dms-select-target.png "MongoDB to Azure Database for CosmosDB - Select target")

    Notice, the **Connection String** will automatically populate with the Key for your Azure Cosmos DB instance.

8. Select **Next: Database setting >>**.

9. On the **Database setting** tab, select the `contentdb` **Source Database** so this database from MongoDB will be migrated to Azure Cosmos DB.

    ![The screenshot shows the Database setting tab with the contentdb source database selected.](media/dms-database-setting.png "Database setting tab")

10. Select **Next: Collection setting >>**.

11. On the **Collection setting** tab, expand the **contentdb** database, and ensure both the **sessions** and **speakers** collections are selected for migration. Also, update the **Throughput (RU/s)** to `400` for both collections.

    ![The screenshot shows the Collection setting tab with both sessions and speakers collections selected with Throughput RU/s set to 400 for both collections.](media/dms-collection-setting.png "Throughput RU")

12. Select **Next: Migration summary >>**.

13. On the **Migration summary** tab, enter `MigrateData` (no spaces) in the **Activity name** field, then select **Start migration** to initiate the migration of the MongoDB data to Azure Cosmos DB.

    ![The screenshot shows the Migration summary is shown with MigrateData entered in the Activity name field.](media/dms-migration-summary.png "Migration summary")

14. The status for the migration activity will be shown. The migration will only take a few seconds to complete. Select **Refresh** to reload the status to ensure it shows a **Status** of **Complete**.

    ![The screenshot shows the MigrateData activity showing the status has completed.](media/dms-migrate-complete.png "MigrateData activity completed")

15. To verify the data was migrated, navigate to the **Cosmos DB Account** for the lab within the Azure Portal, then select the **Data Explorer**. You will see the `speakers` and `sessions` collections listed within the `contentdb` database, and you will be able to explore the documents within.

    ![The screenshot shows the Cosmos DB is open in the Azure Portal with Data Explorer open showing the data has been migrated.](media/dms-confirm-data-in-cosmosdb.png "Cosmos DB is open")

16. We need to create a new index in Cosmos DB for the `startTime` property for documents in the `sessions` collection to avoid errors in our web application. Select **Scale & Settings** for the `sessions` collection and then define a new Index Policy as shown.

   - Set **Defintion** to **startTime** (note: case sensitive).
   - Set **Type** to **Single Field**.
   - Select **Save** to save the new Policy.

   ![The screenshot shows the Cosmos DB Index Policy screen is open in the Azure Portal.](media/sessions-index.png "Cosmos DB edit Index Policy")

## Exercise 3: Deploy the solution to Azure Kubernetes Service

**Duration**: 30 minutes

In this exercise, you will use a combination of the Kubernetes commandline "kubectl" and the Azure Portal to deploy the containerized application to the cluster.

> **Note:** starting with Kubernetes 1.18 the standard Kubernetes dashboard is no longer available by default in AKS. You can choose to install and run it on a Pod if that is your preferred method of management.

### Help references

|                                                        |                                                                                       |
| ------------------------------------------------------ | :-----------------------------------------------------------------------------------: |
| **Description**                                        | **Links**                                                                             |
| Overview of kubectl                                    | https://kubernetes.io/docs/reference/kubectl/overview                                 |
| Kubernetes: Using RBAC Authorization                   | https://kubernetes.io/docs/reference/access-authn-authz/rbac/                         |
| Kubernetes: Services                                   | https://kubernetes.io/docs/concepts/services-networking/service/                      |
| Kubernetes: Connecting Applications with Services      | https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/ |
| Introduction to Helm                                   | https://helm.sh/docs/intro/                                                           |
| Azure Monitor for containers overview                  | https://docs.microsoft.com/azure/azure-monitor/insights/container-insights-overview   |

### Task 1: Deploy a service using the Azure Portal

In this task, you will deploy the API application to the Azure Kubernetes Service cluster using the Azure Portal.

1. We first need to define a Service for our API so that the applicaton is accessible within the cluster. In the AKS blade in the Azure Portal select **Services and ingresses** and on the Services tab select **+ Add**.

    ![This is a screenshot of the Azure Portal for AKS showing adding a Service.](media/2021-02-17_8-22-59.png "Add a Service")

2. In the **Add with YAML** screen, paste following YAML and choose **Add**.

   ```yaml
      apiVersion: v1
      kind: Service
      metadata:
         labels:
            app: api
         name: api
      spec:
         ports:
            - name: api-traffic
               port: 3001
               protocol: TCP
               targetPort: 3001
      selector:
         app: api
      sessionAffinity: None
      type: ClusterIP
   ```

   > Note: you can [download this as a YAML file](lab-files/yaml-templates/ex3-task1-service.yaml) and use the file as your template. 

3. Now select **Workloads** under the **Kubernetes resources** section in the left navigation.

    ![Select workloads under Kubernetes resources.](media/2021-02-16_13-59-15.png "Select workloads under Kubernetes resources")

2. From the Workloads view, with **Deployments** selected (the default) then select **+ Add**.

   ![Selecing + Add to create a deployment.](media/2021-02-16_14-12-50.png "Selecing + Add to create a deployment")

3. In the **Add with YAML** screen that loads paste the following YAML and update [LOGINSERVER] value to match your Azure Container Registry.

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
      labels:
         app: api
      name: api
   spec:
      replicas: 1
      selector:
         matchLabels:
            app: api
   strategy:
      rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
      type: RollingUpdate
   template:
      metadata:
         labels:
            app: api
         name: api
      spec:
         containers:
            - name: api 
               image: [LOGINSERVER].azurecr.io/content-api
               imagePullPolicy: Always
               securityContext:
                  privileged: false
               ports:
                  - containerPort: 3001
                     hostPort: 3001
                     protocol: TCP
               livenessProbe:
                  httpGet:
                     path: /
                     port: 3001
                  initialDelaySeconds: 30
                  periodSeconds: 20
                  timeoutSeconds: 10
                  failureThreshold: 3
               resources:
                  requests:
                     cpu: 1
                     memory: 128Mi
      terminationMessagePath: /dev/termination-log
      terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
   ```

> Note: you can [download this as a YAML file](lab-files/yaml-templates/ex3-task1-deployment.yaml) and use the file as your template. Make sure to update [LOGINSERVER]!

4. Select **Add** to initiate the deployment. This can take a few minutes after which you will see the deployment listed.

   ![Service is showing as unhealthy](media/2021-02-16_14-31-08.png "Service is showing as unhealthy")

5. Select the **api** deployment to open the Deployment, select **Live logs** and then a Pod from the drop-down. After a few moments the live logs should appear.

   ![Service is showing as unhealthy](media/2021-02-16_14-37-57.png "Service is showing as unhealthy")

   > **Note:** if the logs don't display it may be the Pod no longer exists. You can use the **View in Log Analytics** to view historical logs regardless of Pod.

6. If you scroll through the the log you can see it indicates that the content-api application is once again failing because it cannot find a MongoDB api to communicate with. You will resolve this issue by connecting to Cosmos DB.

   ![This screenshot of the Kubernetes management dashboard shows logs output for the api container.](media/2021-02-16_14-39-44.png "MongoDB communication error")

7. In the Azure Portal navigate to your resource group and find your Cosmos DB. Select the Cosmos DB resource to view details.

   ![This is a screenshot of the Azure Portal showing the Cosmos DB among existing resources.](media/Ex2-Task1.9.png "Select CosmosDB resource from list")

8. Under **Quick Start** select the **Node.js** tab and copy the **Node.js 3.0 connection string**.

   ![This is a screenshot of the Azure Portal showing the quick start for setting up Cosmos DB with MongoDB API. The copy button is highlighted.](media/Ex2-Task1.10.png "Capture CosmosDB connection string")

9. Modify the copied connection string by adding the database `contentdb` to the URL, along with a replicaSet of `globaldb`. The resulting connection string should look like the below sample.

   > **Note**: Username and password redacted for brevity.

   ```text
   mongodb://<USERNAME>:<PASSWORD>@fabmedical-<SUFFIX>.documents.azure.com:10255/contentdb?ssl=true&replicaSet=globaldb
   ```

11. You will setup a Kubernetes secret to store the connection string and configure the `content-api` application to access the secret. First, you must base64 encode the secret value. Open your Azure Cloud Shell window and use the following command to encode the connection string and then, copy the output.

    > **Note**: Double quote marks surrounding the connection string are required to successfully produce the required output.

    ```bash
    echo -n "[CONNECTION STRING VALUE]" | base64 -w 0 - | echo $(</dev/stdin)
    ```

    ![This is a screenshot of the Azure cloud shell window showing the command to create the base64 encoded secret.  The output to copy is highlighted.](media/hol-2019-10-18_07-12-13.png "Show encoded secret")

12. Return to the AKS blade in the Azure Portal and select **Configuration** under the **Kubernetes resources** section. Select **Secrets** and choose **+ Add**.

13. In the **Add with YAML** screen, paste following YAML and replace the placeholder with the encoded connection string from your clipboard and choose **Add**. Note that YAML is position sensitive so you must ensure indentation is correct when typing or pasting.

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: cosmosdb
    type: Opaque
    data:
      db: <base64 encoded value>
    ```

> Note: you can [download this as a YAML file](lab-files/yaml-templates/ex3-task1-secret.yaml) and use the file as your template. Make sure to update <base64 encoded value>!


   ![This is a screenshot of the Azure Portal for AKS howing the YAML file for creating a deployment.](media/2021-02-16_14-52-33.png "Upload YAML data")

14. Sort the Secrets list by name and you should now see your new secret displayed. 

    ![This is a screenshot of the Azure Portal for AKS showing secrets.](media/2021-02-16_14-57-18.png "Manage Kubernetes secrets")

15. View the details for the **cosmosdb** secret by selected it in the list.

    ![This is a screenshot of the Azure Portal for AKS showing the value of a secret.](media/2021-02-16_14-59-17.png "View cosmosdb secret")

16. Next, download the API deployment configuration using the following commands in your Azure Cloud Shell window. First we must setup kubectl so it is authorised to talk to our AKS cluster. Replace the resource group and cluster placeholders so they match yours.

   ```bash
   az aks get-credentials--resource-group {resourceGroup} --name {aksClusterName}

   kubectl get -o=yaml deployment api > api.deployment.yml
   ```

17. Edit the downloaded file using cloud shell code editor:

    ```bash
    code api.deployment.yml
    ```

    Add the following environment configuration to the container spec, below the `image` property:

    ```yaml
      env:
      - name: MONGODB_CONNECTION
        valueFrom:
          secretKeyRef:
            name: cosmosdb
            key: db
    ```

    ![This is a screenshot of the Kubernetes management dashboard showing part of the deployment file.](media/Ex2-Task1.17.png "Edit the api.deployment.yml file")

18. Save your changes and close the editor.

    ![This is a screenshot of the code editor save and close actions.](media/Ex2-Task1.17.1.png "Code editor configuration update")

19. Update the api deployment by using `kubectl` to deploy the API.

   ```bash
   kubectl delete deployment api
   kubectl create -f api.deployment.yml
   ```

20. In the Azure Portal return to Live logs (see Step 5). The last log should show as connected to MongoDB.

    ![This is a screenshot of the Kubernetes management dashboard showing logs output.](media/2021-02-16_16-23-27.png "API Logs")

### Task 2: Deploy a service using kubectl

In this task, deploy the web application and expose it to the Internet via a Kubernetes LoadBalancer service. You will be using the Kubernetes command line `kubectl` to complete these steps.

1. Open a **new** Azure Cloud Shell console.

2. Create a text file called `web.deployment.yml` using the Azure Cloud Shell
   Editor.

   ```bash
   code web.deployment.yml
   ```

3. Copy and paste the following text into the editor:

   > **Note**: Be sure to copy and paste only the contents of the code block carefully to avoid introducing any special characters.

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     labels:
         app: web
     name: web
   spec:
     replicas: 1
     selector:
         matchLabels:
           app: web
     strategy:
         rollingUpdate:
           maxSurge: 1
           maxUnavailable: 1
         type: RollingUpdate
     template:
         metadata:
           labels:
               app: web
           name: web
         spec:
           containers:
           - image: [LOGINSERVER].azurecr.io/content-web
             env:
               - name: CONTENT_API_URL
                 value: http://api:3001
             livenessProbe:
               httpGet:
                   path: /
                   port: 3000
               initialDelaySeconds: 30
               periodSeconds: 20
               timeoutSeconds: 10
               failureThreshold: 3
             imagePullPolicy: Always
             name: web
             ports:
               - containerPort: 3000
                 hostPort: 80
                 protocol: TCP
             resources:
               requests:
                   cpu: 1000m
                   memory: 128Mi
             securityContext:
               privileged: false
             terminationMessagePath: /dev/termination-log
             terminationMessagePolicy: File
           dnsPolicy: ClusterFirst
           restartPolicy: Always
           schedulerName: default-scheduler
           securityContext: {}
           terminationGracePeriodSeconds: 30
   ```

   > Note: you can [download this as a YAML file](lab-files/yaml-templates/ex3-task2-deployment.yaml) and use the file as your template. Make sure to update [LOGINSERVER].

4. Update the `[LOGINSERVER]` entry to match the name of your Azure Container Registry Login Server.

5. Select the **...** button and choose **Save**.

   ![In this screenshot of an Azure Cloud Shell editor window, the ... button has been selected and the Save option is highlighted.](media/b4-image62.png "Save Azure Cloud Shell changes")

6. Select the **...** button again and choose **Close Editor**.

   ![In this screenshot of the Azure Cloud Shell editor window, the ... button has been selected and the Close Editor option is highlighted.](media/b4-image63.png "Close Azure Cloud Editor")

7. Create a text file called `web.service.yml` using the Azure Cloud Shell
   Editor.

   ```bash
   code web.service.yml
   ```

8. Copy and paste the following text into the editor:

   > **Note**: Be sure to copy and paste only the contents of the code block carefully to avoid introducing any special characters.

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     labels:
       app: web
     name: web
   spec:
     ports:
       - name: web-traffic
         port: 80
         protocol: TCP
         targetPort: 3000
     selector:
       app: web
     sessionAffinity: None
     type: LoadBalancer
   ```

   > Note: you can [download this as a YAML file](lab-files/yaml-templates/ex3-task2-service.yaml) and use the file as your template.

9. Save changes and close the editor.

10. Type the following command to deploy the application described by the YAML files. You will receive a message indicating the items kubectl has created a web deployment and a web service.

    ```bash
    kubectl create --save-config=true -f web.deployment.yml -f web.service.yml
    ```

    ![In this screenshot of the console, kubectl apply -f kubernetes-web.yaml has been typed and run at the command prompt. Messages about web deployment and web service creation appear below.](media/image93.png "kubectl create application")

11. Return to the AKS blade in the Azure Portal. From the navigation menu, under **Kubernetes resources**, select the **Services and ingresses** view. You should be able to access the website via an external endpoint.

    ![In the Kubernetes management dashboard, Services is selected below Discovery and Load Balancing in the navigation menu. At right are three boxes that display various information about the web service deployment: Details, Pods, and Events.](media/image94.png "Display External Endpoint")

14. In the top navigation, select the `speakers` and `sessions` links.

    ![A screenshot of the web site showing no data displayed.](media/Ex2-Task3.11.png "Web site home page")

### Task 3: Deploy a service using a Helm chart

In this task, you will deploy the **content-web** service using a [Helm](https://helm.sh/) chart to streamline the installing and managing the container-based application on the AKS cluster.

1. From the AKS blade in the Azure Portal, under **Kubernetes resources**, select **Workloads**.

2. Select the `web` Deployment and then choose **Delete**. When prompted, check **Confirm delete** and select **Delete** again.

   ![A screenshot of the Kubernetes management dashboard showing how to delete a deployment.](media/Ex2-Task4.2.png "Kubernetes dashboard web deployments")

3. From the AKS blade in the Azure Portal, under **Kubernetes resources**, select **Services and ingresses**.

4. Select the `web` Service and then choose **Delete**. When prompted, check **Confirm delete** and select **Delete** again.

   ![A screenshot of the Kubernetes management dashboard showing how to delete a deployment.](media/Ex2-Task4.4.png "Kubernetes delete deployment")

5. Open a **new** Azure Cloud Shell.

6. Clone your fabmedical repository (replace URL with URL of your repository):

    ```bash
    git clone https://github.com/USER_NAME/fabmedical.git
    ```

7. We will use the `helm create` command to scaffold out a chart implementation that we can build on. Use the following commands to create a new chart named `web` in a new directory (replace 'fabmedical' with the directory created by your clone):

    ```bash
    cd fabmedical
    cd content-web
    mkdir charts
    cd charts
    helm create web
    ```

8. We now need to update the generated scaffold to match our requirements. We will first update the file named `values.yaml`.

    ```bash
    cd web
    code values.yaml
    ```

9. Search for the `image` definition and update the values so that they match the following (remember to replace [LOGINSERVER]):

    ```yaml
    image:
      repository: [LOGINSERVER].azurecr.io/content-web
      pullPolicy: Always
    ```

10. Search for `nameOverride` and `fullnameOverride` entries and update the values so that they match the following:

    ```yaml
    nameOverride: "web"
    fullnameOverride: "web"
    ```

11. Search for the `service` definition and update the values so that they match the following:

    ```yaml
    service:
      type: LoadBalancer
      port: 80
    ```

12. Search for the `resources` definition and update the values so that they match the following. You are removing the curly braces and adding the `requests` (make sure to remove the {} characters after the `resource:` node):

    ```yaml
    resources:
      # We usually recommend not to specify default resources and to leave this as a conscious
      # choice for the user. This also increases chances charts run on environments with little
      # resources, such as Minikube. If you do want to specify resources, uncomment the following
      # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
      # limits:
      #  cpu: 100m
      #  memory: 128Mi
      requests:
        cpu: 1000m
        memory: 128Mi
    ```

13. Save changes and close the editor.

14. We will now update the file named `Chart.yaml`.

    ```bash
    code Chart.yaml
    ```

15. Search for the `appVersion` entry and update the value so that it matches the following:

    ```yaml
    appVersion: latest
    ```

16. We will now update the file named `deployment.yaml`.

    ```bash
    cd templates
    code deployment.yaml
    ```

17. Search for the `metadata` definition and update the values so that they match the following. You are replacing the line under annotations:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      (...)
    spec:
      (...)
      template:
        metadata:
          (...)
          annotations:
            rollme: {{ randAlphaNum 5 | quote }}
    ```

18. Search for the `containers` definition and update the values so that they match the following. You are changing the `containerPort`, `livenessProbe` port and adding the `env` variable:

    ```yaml
    containers:
      - name: {{ .Chart.Name }}
        securityContext:
          {{- toYaml .Values.securityContext | nindent 12 }}
        image: "{{ .Values.image.repository }}:{{ .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
          - name: http
            containerPort: 3000
            protocol: TCP
        env:
          - name: CONTENT_API_URL
            value: http://api:3001
        livenessProbe:
          httpGet:
            path: /
            port: 3000
    ```

19. Save changes and close the editor.

20. We will now update the file named `service.yaml`.

    ```bash
    code service.yaml
    ```

21. Search for the `ports` definition and update the values so that they match the following:

    ```yaml
    ports:
      - port: {{ .Values.service.port }}
        targetPort: 3000
        protocol: TCP
        name: http
    ```

22. Save changes and close the editor.

23. The chart is now setup to deploy our web container. Type the following command to deploy the application described by the Helm chart. You will receive a message indicating that helm has created a web deployment and a web service.

    ```bash
    cd ../..
    helm install web ./web
    ```

    ![In this screenshot of the console, helm install web ./web has been typed and run at the command prompt. Messages about web deployment and web service creation appear below.](media/Ex2-Task4.24.png "Helm web deployment messages")

24. Return to the browser where you have the Azure Portal open. From the navigation menu, select **Services and ingresses**. You will see the web service deploying which deployment can take a few minutes. When it completes, you should be able to access the website via an external endpoint.

    ![In the AKS Services and ingresses blade in the Azure Portal showing the web service selected.](media/image94.png "Web service endpoint")

25. Select the speakers and sessions links and check that content is displayed for each.

    ![A screenshot of the web site showing no data displayed.](media/Ex2-Task3.11.png "Web site home page")

26. We will now commit our Helm chart to our GitHubs repository. Execute the following commands in the root folder of your 'fabmedical' clone:

    ```bash
    git add content-web/charts/
    git commit -m "Helm chart added."
    git push
    ```

### Task 4: Configure Continuous Delivery to the Kubernetes Cluster

In this task, you will use GitHub Actions to automate the process for deploying the web image to the AKS cluster using Helm. You will update the GitHub Action and configure a job so that when new images are pushed to the Azure Container Registry (ACR), the pipeline deploys the image to the AKS cluster. ACR can also host Helm charts.

1. On your lab VM perform the following steps to edit the GitHub Action for the conent-web container. If you setup Visual Studio Code remote development you can also use that. Open or navigate to the folder where you originally created your fabmedical git repository (Task 7 of the 'Before the HOL' exercise).

   ```bash
   git pull
   cd .github/workflows
   vi content-web.yml
   ```

2. You will add a second job to the bottom of the `content-web.yml` workflow. Paste the following at the end of the file:

    > **Note**: Be careful to check your indenting when pasting. The `build-and-push-helm-chart` node should be indented with 2 spaces and line up with the node for the `build-and-publish-docker-image` job.

    ```yaml
      build-and-push-helm-chart:
        name: Build and Push Helm Chart
        runs-on: ubuntu-latest
        needs: [build-and-publish-docker-image]
        steps:
        # Checkout the repo
        - uses: actions/checkout@master

        - name: Helm Install
          uses: azure/setup-helm@v1

        - name: Helm Repo Add
          run: |
            helm repo add ${{ env.containerRegistryName }} https://${{ env.containerRegistry }}/helm/v1/repo --username ${{ secrets.ACR_USERNAME }} --password ${{ secrets.ACR_PASSWORD }}
          env:
            HELM_EXPERIMENTAL_OCI: 1

        - name: Helm Chart Save
          run: |
            cd ./content-web/charts/web

            helm chart save . content-web:v${{ env.tag }}
            helm chart save . ${{ env.containerRegistry }}/helm/content-web:v${{ env.tag }}

            # list out saved charts
            helm chart list
          env:
            HELM_EXPERIMENTAL_OCI: 1

        - name: Helm Chart Push
          run: |
            helm registry login ${{ env.containerRegistry }} --username ${{ secrets.ACR_USERNAME }} --password ${{ secrets.ACR_PASSWORD }}
            helm chart push ${{ env.containerRegistry }}/helm/content-web:v${{ env.tag }}
          env:
            HELM_EXPERIMENTAL_OCI: 1
    ```

   > Note: you can download the GitHub Actions file below after step 7.

3. Save the file.

4. In the Azure Cloud Shell, use the following command to output the `/.kube/config` file that contains the credentials for authenticating with Azure Kubernetes Service. These credentials were retrieved previously and will also be needed by GitHub Actions to deploy to AKS. Then copy the contents of the file.

    ```bash
    cat ~/.kube/config
    ```

5. In GitHub, return to the **Fabmedical** repository, select the **Settings** tab, select **Secrets** from the left menu, then select the **New secret** button.

6. Create a new GitHub Secret with the Name of `KUBECONFIG` and paste in the contents of the `~/.kube/config` file that was previously copied.

    ![The screenshot displays the KUBECONFIG secret](media/2020-08-25-22-34-04.png "Edit KUBECONFIG secret")

7. On your lab VM edit the `content-web.yml` workflow and paste the following at the end of the file.

    > **Note**: Be careful to check your indenting when pasting. The `aks-deployment` node should be indented with 2 spaces and line up with the node for the `build-and-push-helm-chart` job.

    ```yaml
      aks-deployment:
        name: AKS Deployment
        runs-on: ubuntu-latest
        needs: [build-and-publish-docker-image,build-and-push-helm-chart]
        steps:
        # Checkout the repo
        - uses: actions/checkout@master

        - name: Helm Install
          uses: azure/setup-helm@v1

        - name: kubeconfig
          run: echo "${{ secrets.KUBECONFIG }}" >> kubeconfig

        - name: Helm Repo Add
          run: |
            helm repo add ${{ env.containerRegistry }} https://${{ env.containerRegistry }}/helm/v1/repo --username ${{ secrets.ACR_USERNAME }} --password ${{ secrets.ACR_PASSWORD }}
            helm repo update
          env:
            HELM_EXPERIMENTAL_OCI: 1

        - name: Helm Upgrade
          run: |
            helm registry login ${{ env.containerRegistry }} --username ${{ secrets.ACR_USERNAME }} --password ${{ secrets.ACR_PASSWORD }}
            helm chart pull ${{ env.containerRegistry }}/helm/content-web:v${{ env.tag }}
            helm chart export ${{ env.containerRegistry }}/helm/content-web:v${{ env.tag }} --destination ./upgrade
            helm upgrade web ./upgrade/web
          env:
            KUBECONFIG: './kubeconfig'
            HELM_EXPERIMENTAL_OCI: 1
    ```

   > Note: you can [download this as a YAML file](lab-files/yaml-templates/ex3-task4-actions.yaml) and use the file as your template. Make sure to update [LOGINSERVER].

8. Save the file.

9. Commit your changes

   ```bash
   cd ..
   git pull
   git add --all
   git commit -m "Deployment update."
   git push
   ```

10. Switch back to GitHub.

11. On the **content-web** workflow, select **Run workflow** and manually trigger the workflow to execute.

    ![The content-web Action is shown with the Actions, content-web, and Run workflow links highlighted.](media/2020-08-25-15-38-06.png "content-web workflow")

12. Selecting the currently running workflow will display its status.

    ![The screenshot shows workflow is running and the current status.](media/2020-08-25-22-15-39.png "Workflow is running")

### Task 5: Review Azure Monitor for Containers

In this task, you will access and review the various logs and dashboards made available by Azure Monitor for Containers.

1. From the Azure Portal, select the resource group you created named `fabmedical-SUFFIX`, and then select your `Kubernetes Service` Azure resource.

   ![In this screenshot, the resource group was previously selected and the AKS cluster is selected.](media/Ex2-Task8.1.png "Select fabmedical resource group")

2. From the Monitoring blade, select **Insights**.

   ![In the Monitoring blade, Insights is highlighted.](media/Ex2-Task8.2.png "Select Insights link")

3. Review the various available dashboards and a deeper look at the various metrics and logs available on the Cluster, Nodes, Controllers, and deployed Containers.

   ![In this screenshot, the dashboards and blades are shown. Cluster metrics can be reviewed.](media/Ex2-Task8.3.png "Review the dashboard metrics")

4. To review the Containers dashboards and see more detailed information about each container, select the **Containers** tab.

   ![In this screenshot, the various containers information is shown.](media/monitor_1.png "View containers data")

5. Now filter by container name and search for the **web** containers, you will see all the containers created in the Kubernetes cluster with the pod names. You can compare the names with those in the kubernetes dashboard.

   ![In this screenshot, the containers are filtered by container named web.](media/monitor_3.png "Filter data by container and web")

6. By default, the CPU Usage metric will be selected displaying all cpu information for the selected container, to switch to another metric open the metric dropdown list and select a different metric.

   ![In this screenshot, the various metric options are shown.](media/monitor_2.png "Filter by CPU usage")

7. Upon selecting any pod, all the information related to the selected metric will be displayed on the right panel, and that would be the case when selecting any other metric, the details will be displayed on the right panel for the selected pod.

   ![In this screenshot, the pod cpu usage details are shown.](media/monitor_4.png "POD CPU details")

8. To display the logs for any container simply select it and view the right panel and you will find the **View container logs** option which will list all logs for this specific container.

   ![In the View in Analytics dropdown, the View container logs item is selected.](media/monitor_5.png "View container logs menu option")

   ![The container logs are displayed based on a query entered in the query window.](media/monitor_6.png "Container logs")

9. For each log entry you can display more information by expanding the log entry to view the below details.

   ![The container log query results are displayed, one log entry is expanded in the results view with its details shown.](media/monitor_7.png "Expand the results")

## Exercise 4: Scale the application and test High Availability

**Duration**: 20 minutes

At this point, you have deployed a single instance of the web and API service containers. In this exercise, you will increase the number of container instances for the web service and scale the front-end on the existing cluster.

### Help references

|                                            |                                                                    |
| ------------------------------------------ | :----------------------------------------------------------------: |
| **Description**                            | **Links**                                                          |
| Overview of kubectl                        | https://kubernetes.io/docs/reference/kubectl/overview              |
| Kubernetes: Services                       | https://kubernetes.io/docs/concepts/services-networking/service/   |

### Task 1: Increase service instances from the Azure Portal

In this task, you will increase the number of instances for the API deployment in the AKS Azure Portal blade. While it is deploying, you will observe the changing status.

1. In the AKS blade in the Azure Portal select **Workloads** and then select the **API** deployment.

2. Select **YAML** in the window that loads and scroll down until you find **replicas**. Change the number of replicas to **2**, and then select **Review + save**. When prompted, check **Confirm manifest change** and select **Save**.

   ![In the edit YAML dialog, 2 is entered in the desired number of replicas.](media/k8s-deploy-scale.png "Setting replicas to 2")

   > **Note**: If the deployment completes quickly, you may not see the deployment Waiting states in the portal, as described in the following steps.

3. From the Replica Set view for the API, you will see it is now deploying and that there is one healthy instance and one pending instance.

   ![Replica Sets is selected under Workloads in the navigation menu on the left, and at right, Pods status: 1 pending, 1 running is highlighted. Below that, a red arrow points at the API deployment in the Pods box.](media/image117.png "View replica details")

4. From the navigation menu, select **Workloads**. Note that the api Deployment has an alert and shows a pod count 1 of 2 instances (shown as `1/2`).

   ![In the Deployments box, the api service is highlighted with a grey timer icon at left and a pod count of 1/2 listed at right.](media/image118.png "View api active pods")

   > **Note**: If you receive an error about insufficient CPU that is OK. We will see how to deal with this in the next Task (Hint: you can use the **Insights** option in the AKS Azure Portal to review the **Node** status and view the Kubernetes event logs).

5. From the Navigation menu, select **Workloads**. From this view, note that the health overview in the right panel of this view. You will see the following:

   - One Deployment and one Replica Set are each healthy for the web service.

   - The api Deployment and Replica Set are in a warning state.

   - Two pods are healthy in the 'default' namespace.

6. Open the Contoso Neuro Conference web application. The application should still work without errors as you navigate to Speakers and Sessions pages.

   - Navigate to the `/stats` page. You will see information about the hosting environment including:

     - **webTaskId:** The task identifier for the web service instance.

     - **taskId:** The task identifier for the API service instance.

     - **hostName:** The hostname identifier for the API service instance.

     - **pid:** The process id for the API service instance.

     - **mem:** Some memory indicators returned from the API service instance.

     - **counters:** Counters for the service itself, as returned by the API service instance.

     - **uptime:** The up time for the API service.

### Task 2: Resolve failed provisioning of replicas

In this task, you will resolve the failed API replicas. These failures occur due to the clusters' inability to meet the requested resources.

1. In the AKS blade in the Azure Portal select **Workloads** and then select the **API** deployment. Select the **YAML** navigation item.

2. In the **YAML** screen scroll down and change the following items:

   - Modify **ports** and remove the **hostPort**. Two Pods cannot map to the same host port.

      ```yaml
      ports:
         - containerPort: 3001
         protocol: TCP
      ```

   - Modify the **cpu** and set it to **100m**. CPU is divided between all Pods on a Node.

      ```yaml
      resources:
         requests:
            cpu: 100m
            memory: 128Mi
      ```

   Select **Review + save** and, when prompted, confirm the changes and select **Save**.

   ![In the edit YAML dialog, showing two changes required.](media/2021-02-17_10-26-33.png "Modify deployment manifest")

3. Return to the **Workloads** main view on the AKS Azure Portal and you will now see that the Deployment is healthy with two Pods operating.

   ![In the Workload view with the API deployment highlighted.](media/2021-02-17_10-48-19.png "API deployment is now healthy")

### Task 3: Restart containers and test High Availability

In this task, you will restart containers and validate that the restart does not impact the running service.

1. Open the sample web application and navigate to the "Stats" page as shown.

   ![The Stats page is visible in this screenshot of the Contoso Neuro web application.](media/image123.png "Contoso web task details")

2. In the AKS blade in the Azure Portal open the api Deployment and increase the required replica count to `4`. Use the same process as Exercise 4, Task 1.

   ![In the left menu the Deployments item is selected. The API deployment is highlighted in the Deployments list box.](media/2021-02-17_10-53-46.png "API pod deployments")

3. After a few moments you will find that the API deployment is now running 4 replicas successfully.

4. Return to the browser tab with the web application stats page loaded. Refresh the page over and over. You will not see any errors, but you will see the api host name change between the two api pod instances periodically. The task id and pid might also change between the two api pod instances.

   ![On the Stats page in the Contoso Neuro web application, two different api host name values are highlighted.](media/image126.png "View web task hostname")

6. After refreshing enough times to see that the `hostName` value is changing, and the service remains healthy, you can open the **Replica Sets** view for the API in the Azure Portal.

7. On this view you can see the hostName value shown in the web application stats page matches the pod names for the pods that are running.

   ![Viewing replica set in the Azure Portal.](media/image128.png "Viewing replica set in the Azure Portal")

8. Select two of the Pods at random and choose **Delete**.

   ![The context menu for a pod in the pod list is expanded with the Delete item selected.](media/2021-02-17_11-04-35.png "Delete running pod instance")

9. Kubernetes will launch new Pods to meet the required replica count. Depending on your view you may see the old instances Terminating and new instances being Created.

   ![The first row of the Pods box is highlighted, and the pod has a green check mark and is running.](media/2021-02-17_11-05-59.png "API Pods changing state")

10. Return to the API Deployment and scale it back to `1` replica. See Step 2 above for how to do this if you are unsure.

11. Return to the sample web site's stats page in the browser and refresh while Kubernetes is scaling down the number of Pods. You will notice that only one API host name shows up, even though you may still see several running pods in the API replica set view. Even though several pods are running, Kubernetes will no longer send traffic to the pods it has selected to terminate. In a few moments, only one pod will show in the API Replica Set view.

    ![Replica Sets is selected under Workloads in the navigation menu on the left. On the right are the Details and Pods boxes. Only one API host name, which has a green check mark and is listed as running, appears in the Pods box.](media/image131.png "View replica details")

### Task 4: Configure Cosmos DB Autoscale

In this task, you will setup Autoscale on Azure Cosmos DB.

1. In the Azure Portal, navigate to the `fabmedical-[SUFFIX]` **Azure Cosmos DB Account**.

2. Select **Data Explorer**. 

3. Within **Data Explorer**, expand the `contentdb` database, then expand the `sessions` collection.

4. Under the `sessions` collection, select **Scale & Settings**.

5. On the **Scale & Settings**, select **Autoscale** for the **Throughput** setting under **Scale**.

    ![The screenshot displays Cosmos DB Scale and Settings tab with Autoscale selected](media/cosmosdb-autoscale.png "CosmosDB collection scale and settings")

6. Select **Save**.

7. Perform the same task to enable **Autoscale** Throughput on the `speakers` collection.

### Task 5: Test Cosmos DB Autoscale

In this task, you will run a performance test script that will test the Autoscale feature of Azure Cosmos DB so you can see that it will now scale greater than 400 RU/s.

1. In the Azure Portal, navigate to the `fabmedical-[SUFFIX]` **Cosmos DB Account**.

2. Select **Connection String** under **Settings**.

3. On the **Connection String** pane, copy the **HOST**, **USERNAME**, and **PRIMARY PASSWORD** values. Save these for use later.

    ![The Cosmos DB account Connection String pane with the fields to copy highlighted.](media/cosmos-connection-string-pane.png "View CosmosDB connection string")

4. On your lab VM navigate to the location you cloned your `fabmedical` repository.

    ```bash
    cd ~/fabmedical
    ```

6. Open the `perftest.sh` script for editing in Vim (or Visual Studio Code remote if you have it installed)

    ```bash
    vi perftest.sh
    ```

7. There are several variables declared at the top of the `perftest.sh` script. Modify the **host**, **username**, and **password** variables by setting their values to the corresponding Cosmos DB Connection String values that were copied previously.

    ![The screenshot shows Vim with perftest.sh file open and variables set to Cosmos DB Connection String values.](media/cosmos-perf-test-variables.png "Modify the connection information in Vim")

8. Save the file and exit Vim.

9. Run the following command to execute the `perftest.sh` script to run a small load test against Cosmos DB. This script will consume RU's in Cosmos DB by inserting many documents into the Sessions container.

    ```bash
    bash ./perftest.sh
    ```

    > **Note:** The script will take a minute to complete executing.

10. Once the script has completed, navigate back to the **Cosmos DB account** in the Azure portal.

11. Reset the script to the one from git by running the following command. This ensures you don't commit your Cosmos DB credentials to source control.

    ```bash
    git checkout -- perftest.sh
    ```

12. Scroll down on the **Overview** pane of the **Cosmos DB account** blade, and locate the **Request Charge** graph.

    > **Note:** It may take 2 - 5 minutes for the activity on the Cosmos DB collection to appear in the activity log. Wait a couple minutes and then refresh the pane if the recent Request charge doesn't show up right now.

13. Notice that the **Request charge** now shows there was activity on the **Cosmos DB account** that exceeded the 400 RU/s limit that was previously set before Autoscale was turned on.

    ![The screenshot shows the Cosmos DB request charge graph showing recent activity from performance test](media/cosmos-request-charge.png "Recent CosmosDB activity graph")

## Exercise 5: Working with services and routing application traffic

**Duration**: 1 hour

In the previous exercise with Kubernetes, we removed restrictions to the scale properties of the service. In this exercise, you will configure the api deployment to create pods that use dynamic port mappings to eliminate the port resource constraint during scale activities.

Kubernetes services can discover the ports assigned to each pod, allowing you to run multiple instances of the pod on the same agent node --- something that is not possible when you configure a specific static port (such as the 3001 mapping for the API service that we put in place in the last Exercise).

### Help references

|                                            |                                                                    |
| ------------------------------------------ | :----------------------------------------------------------------: |
| **Description**                            | **Links**                                                          |
| Use a public Standard Load Balancer in Azure Kubernetes Service (AKS) | https://docs.microsoft.com/azure/aks/load-balancer-standard              |
| Create an HTTPS ingress controller on Azure Kubernetes Service (AKS)  | https://docs.microsoft.com/azure/aks/ingress-tls |
| What is Application Insights?                                         | https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview |
| Kubernetes: Ingress                                                   | https://kubernetes.io/docs/concepts/services-networking/ingress/         |
| Kubernetes: Ingress Controllers                                       | https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/ |
| Kubernetes: cert-manager                                              | https://cert-manager.io/docs/installation/kubernetes/ |


### Task 1: Update an external service to support dynamic discovery with a load balancer

In this task, you will update the web service so that it supports dynamic discovery through an Azure load balancer.

1. From AKS **Kubernetes resources** menu, select **Deployments** under **Workloads**. From the list select the **web** deployment.

2. Select **YAML**, then select the **JSON** tab.

3. First locate the replicas node and update the required count to `4`.

3. Next, scroll to the web containers spec as shown in the screenshot. Remove the hostPort entry for the web container's port mapping.

   ![This is a screenshot of the Edit a Deployment dialog box with various displayed information about spec, containers, ports, and env. The ports node, containerPort: 3001 and protocol: TCP are highlighted.](media/image140.png "Remove web container hostPort entry")

4. Select **Review + save** and then confirm the change and **Save**.

6. Check the status of the scale out by refreshing the web deployment's view. From the navigation menu, select **Pods** from under Workloads. Select the **web** pods. From this view, you should see an error like that shown in the following screenshot.

   ![Deployments is selected under Workloads in the navigation menu on the left. On the right are the Details and New Replica Set boxes. The web deployment is highlighted in the New Replica Set box, indicating an error.](media/image141.png "View Pod deployment events")

Like the API deployment, the web deployment used a fixed _hostPort_, and your ability to scale was limited by the number of available agent nodes. However, after resolving this issue for the web service by removing the _hostPort_ setting, the web deployment is still unable to scale past two pods due to CPU constraints. The deployment is requesting more CPU than the web application needs, so you will fix this constraint in the next task.

### Task 2: Adjust CPU constraints to improve scale

In this task, you will modify the CPU requirements for the web service so that it can scale out to more instances.

1. Re-open the JSON view for the web deployment and then find the **cpu** resource requirements for the web container. Change this value to `125m`.

   ![This is a screenshot of the Edit a Deployment dialog box with various displayed information about ports, env, and resources. The resources node, with cpu: 125m selected, is highlighted.](media/image142.png "Change cpu value")

4. Select **Review + save**, confirm the change and then select **Save** to update the deployment.

5. From the navigation menu, select **Replica Sets** under **Workloads**. From the view's Replica Sets list select the web replica set.

6. When the deployment update completes, four web pods should be shown in running state.

   ![Four web pods are listed in the Pods box, and all have green check marks and are listed as Running.](media/image143.png "Four pods running")

7. Return to the browser tab with the sample web application loaded. Refresh the stats page at /stats to watch the display update to reflect the different api pods by observing the host name refresh.

### Task 3: Perform a rolling update

In this task, you will edit the web application source code to add Application Insights and update the Docker image used by the deployment. Then you will perform a rolling update to demonstrate how to deploy a code change.

1. Execute this command in Azure Cloud Shell to retrieve the instrumentation key for the `content-web` Application Insights resource:

   ```bash
   az resource show -g fabmedical-[SUFFIX] -n content-web --resource-type "Microsoft.Insights/components" --query properties.InstrumentationKey -o tsv
   ```

   Copy this value. You will use it later.

   > **Note:** if you have a blank result check that the command you issued refers to the right resource.

2. On your lab VM update your fabmedical repository files by pulling the latest changes from the git repository:

   ```bash
   cd ~/fabmedical/content-web
   git pull
   ```

3. Install support for Application Insights.

   ```bash
   npm install applicationinsights --save
   ```

4. Edit the `app.js` file using Vim or Visual Studio Code remote and add the following lines immediately after `express` is instantiated on line 6:

   ```javascript
   const appInsights = require("applicationinsights");
   appInsights.setup("[YOUR APPINSIGHTS KEY]");
   appInsights.start();
   ```

   ![A screenshot of the code editor showing updates in context of the app.js file](media/hol-2019-10-02_12-33-29.png "AppInsights updates in app.js")

5. Save changes and close the editor.

6. Push these changes to your repository so that GitHub Actions CI will build and deploy a new Container image.

   ```bash
   git add .
   git commit -m "Added Application Insights"
   git push
   ```

7. Visit the `content-web` Action for your GitHub Fabmedical repository and see the new Image being deployed into your Kubernetes cluster.

8. While this update runs, return the Azure Portal in the browser.

9. From the navigation menu, select **Replica Sets** under **Workloads**. From this view, you will see a new replica set for the web, which may still be in the process of deploying (as shown below) or already fully deployed.

    ![At the top of the list, a new web replica set is listed as a pending deployment in the Replica Set box.](media/image144.png "Pod deployment is in progress")

11. While the deployment is in progress, you can navigate to the web application and visit the stats page at `/stats`. Refresh the page as the rolling update executes. Observe that the service is running normally, and tasks continue to be load balanced.

    ![On the Stats page, the hostName is highlighted.](media/image145.png "On Stats page hostName is displayed")

### Task 4: Configure Kubernetes Ingress

In this task you will setup a Kubernetes Ingress using an [nginx proxy server](https://nginx.org/en/) to take advantage of path-based routing and TLS termination.

1. Within the Azure Cloud Shell, run the following command to add the nginx stable Helm repository:

    ```bash
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    ```

2. Update your helm package list.

   ```bash
   helm repo update
   ```

   > **Note**: If you get a "no repositories found." error, then run the following command. This will add back the official Helm "stable" repository.
   > ```
   > helm repo add stable https://charts.helm.sh/stable 
   > ```

3. Create a namespace in Kubernetes to install the Ingress resources.

    ```bash
    kubectl create namespace ingress-demo
    ```

3. Install the Ingress Controller resource to handle ingress requests as they come in. The Ingress Controller will receive a public IP of its own on the Azure Load Balancer and be able to handle requests for multiple services over port 80 and 443.

   ```bash
   helm install nginx-ingress ingress-nginx/ingress-nginx \
    --namespace ingress-demo \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set controller.admissionWebhooks.patch.nodeSelector."beta\.kubernetes\.io/os"=linux
   ```

4. In the Azure Portal under **Services and ingresses** copy the IP Address for the **External IP** for the `nginx-ingress-RANDOM-nginx-ingress` service.

   ![A screenshot of the Kubernetes management dashboard showing the ingress controller settings.](media/Ex4-Task5.5.png "Copy ingress controller settings")

    > **Note**: It could take a few minutes to refresh, alternately, you can find the IP using the following command in Azure Cloud Shell.
    >
    > ```bash
    > kubectl get svc --namespace ingress-demo
    > ```
    >
   ![A screenshot of Azure Cloud Shell showing the command output.](media/Ex4-Task5.5a.png "View the ingress controller LoadBalancer")

6. Open the [Azure Portal Resource Groups blade](https://portal.azure.com/?feature.customPortal=false#blade/HubsExtension/BrowseResourceGroups) and locate the Resource Group that was automatically created to host the Node Pools for AKS. It will have the naming format of `MC_fabmedical-[SUFFIX]_fabmedical-[SUFFIX]_[REGION]`.

7. Within the Azure Cloud Shell, create a script to update the public DNS name for the ingress external IP.

   ```bash
   code update-ip.sh
   ```

   Paste the following as the contents. Be sure to replace the following placeholders in the script:

   - `[INGRESS PUBLIC IP]`: Replace this with the IP Address copied from step 5.
   - `[AKS NODEPOOL RESOURCE GROUP]`: Replace with the name of the Resource Group copied from step 6.
   - `[SUFFIX]`: Replace this with the same SUFFIX value used previously for this lab.
   
   ```bash
   #!/bin/bash

   # Public IP address
   IP="[INGRESS PUBLIC IP]"

   # Resource Group that contains AKS Node Pool
   KUBERNETES_NODE_RG="[AKS NODEPOOL RESOURCE GROUP]"

   # Name to associate with public IP address
   DNSNAME="fabmedical-[SUFFIX]-ingress"

   # Get the resource-id of the public ip
   PUBLICIPID=$(az network public-ip list --resource-group $KUBERNETES_NODE_RG --query "[?ipAddress!=null]|[?contains(ipAddress, '$IP')].[id]" --output tsv)

   # Update public ip address with dns name
   az network public-ip update --ids $PUBLICIPID --dns-name $DNSNAME
   ```

   ![A screenshot of cloud shell editor showing the updated IP and SUFFIX values.](media/Ex4-Task5.6.png "Update the IP and SUFFIX values")

7. Save changes and close the editor.

8. Run the update script.

   ```bash
   bash ./update-ip.sh
   ```

9. Verify the IP update by visiting the URL in your browser.

   > **Note**: It is normal to receive a 404 message at this time.

   ```text
   http://fabmedical-[SUFFIX]-ingress.[AZURE-REGION].cloudapp.azure.com/
   ```

   ![A screenshot of the fabmedical browser URL.](media/Ex4-Task5.9.png "fabmedical browser URL")

10. Use helm to install `cert-manager`, a tool that can provision SSL certificates automatically from letsencrypt.org.

    ```bash
    kubectl create namespace cert-manager

    kubectl label namespace cert-manager cert-manager.io/disable-validation=true

    kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.1/cert-manager.yaml
    ```

11. Cert manager will need a custom ClusterIssuer resource to handle requesting SSL certificates.

    ```bash
    code clusterissuer.yml
    ```

    The following resource configuration should work as is:

    ```yaml
    apiVersion: cert-manager.io/v1
    kind: ClusterIssuer
    metadata:
      name: letsencrypt-prod
    spec:
      acme:
        # The ACME server URL
        server: https://acme-v02.api.letsencrypt.org/directory
        # Email address used for ACME registration
        email: user@fabmedical.com
        # Name of a secret used to store the ACME account private key
        privateKeySecretRef:
          name: letsencrypt-prod
        # Enable HTTP01 validations
        solvers:
        - http01:
            ingress:
              class: nginx
    ```

      > Note: you can [download this as a YAML file](lab-files/yaml-templates/ex5-task4-certs.yaml) and use the file as your template.

12. Save changes and close the editor.

13. Create the issuer using `kubectl`.

    ```bash
    kubectl create --save-config=true -f clusterissuer.yml
    ```

14. Now you can create a certificate object.

    > **Note**:
    >
    > Cert-manager might have already created a certificate object for you using ingress-shim.
    >
    > To verify that the certificate was created successfully, use the `kubectl describe certificate tls-secret` command.
    >
    > If a certificate is already available, skip to step 16.

    ```bash
    code certificate.yml
    ```

    Use the following as the contents and update the `[SUFFIX]` and `[AZURE-REGION]` to match your ingress DNS name

    ```yaml
    apiVersion: cert-manager.io/v1
    kind: Certificate
    metadata:
      name: tls-secret
    spec:
      secretName: tls-secret
      dnsNames:
        - fabmedical-[SUFFIX]-ingress.[AZURE-REGION].cloudapp.azure.com
      issuerRef:
        name: letsencrypt-prod
        kind: ClusterIssuer
    ```

      > Note: you can [download this as a YAML file](lab-files/yaml-templates/ex5-task4-cert.yaml) and use the file as your template. Make sure to update the placeholders.

15. Save changes and close the editor.

16. Create the certificate using `kubectl`.

    ```bash
    kubectl create --save-config=true -f certificate.yml
    ```

    > **Note**: To check the status of the certificate issuance, use the `kubectl describe certificate tls-secret` command and look for an _Events_ output similar to the following:
    >
    > ```text
    > Type    Reason         Age   From          Message
    > ----    ------         ----  ----          -------
    > Normal  Generated           38s   cert-manager  Generated new private key
    > Normal  GenerateSelfSigned  38s   cert-manager  Generated temporary self signed certificate
    > Normal  OrderCreated        38s   cert-manager  Created Order resource "tls-secret-3254248695"
    > Normal  OrderComplete       12s   cert-manager  Order "tls-secret-3254248695" completed successfully
    > Normal  CertIssued          12s   cert-manager  Certificate issued successfully
    > ```

    It can take between 5 and 30 minutes before the tls-secret becomes available. This is due to the delay involved with provisioning a TLS cert from letsencrypt.

17. Now you can create an ingress resource for the content applications.

   ```bash
   code content.ingress.yml
   ```

    Use the following as the contents and update the `[SUFFIX]` and `[AZURE-REGION]` to match your ingress DNS name:

   ```yaml
   apiVersion: networking.k8s.io/v1beta1
   kind: Ingress
   metadata:
      name: content-ingress
      annotations:
         kubernetes.io/ingress.class: nginx
         nginx.ingress.kubernetes.io/rewrite-target: /$1
         nginx.ingress.kubernetes.io/use-regex: "true"
         nginx.ingress.kubernetes.io/ssl-redirect: "false"
         cert-manager.io/cluster-issuer: letsencrypt-prod
   spec:
      tls:
      - hosts:
         - fabmedical-[SUFFIX]-ingress.[AZURE-REGION].cloudapp.azure.com
         secretName: tls-secret
      rules:
         - host: fabmedical-[SUFFIX]-ingress.[AZURE-REGION].cloudapp.azure.com
         http:
            paths:
            - path: /(.*)
               backend:
                  serviceName: web
                  servicePort: 80
            - path: /content-api/(.*)
               backend:            
                  serviceName: api
                  servicePort: 3001
   ```
      > Note: you can [download this as a YAML file](lab-files/yaml-templates/ex5-task4-ingress.yaml) and use the file as your template. Make sure to update the placeholders.

18. Save changes and close the editor.

19. Create the ingress using `kubectl`.

    ```bash
    kubectl create --save-config=true -f content.ingress.yml
    ```

20. Refresh the ingress endpoint in your browser. You should be able to visit the speakers and sessions pages and see all the content.

21. Visit the API directly, by navigating to `/content-api/sessions` at the ingress endpoint.

    ![A screenshot showing the output of the sessions content in the browser.](media/Ex4-Task5.19.png "Content api sessions")

22. Test TLS termination by visiting both services again using `https`.

    > It can take between 5 and 30 minutes before the SSL site becomes available. This is due to the delay involved with provisioning a TLS cert from letsencrypt.

### Task 5: Multi-region Load Balancing with Traffic Manager

In this task, you will setup Azure Traffic Manager as a multi-region load balancer. This will enable you to provision an AKS instance of the app in a secondary Azure region with load balancing between the two regions.

1. Open the [Azure Portal Create Traffic Manager profile Blade](https://portal.azure.com/#create/Microsoft.TrafficManagerProfile).

2. On the **Create Traffic Manager profile** blade, enter the following values, then select **Create**.

    - Name: `fabmedical-[SUFFIX]'
    - Routing Method: **Performance**
    - Resource Group: `fabmedical-[SUFFIX]`

    ![The screenshot shows the Create Traffic Manager profile blade with all values entered.](media/tm-create.png "Create Traffic Manager profile configuration")

4. Once deployment completes, navigate to the newly created `fabmedical-[SUFFIX]` **Traffic Manager profile**.

5. On the **Traffic Manager profile** blade, select **Endpoints** under **Settings** on the left navigation.

6. On the **Endpoints** pane, select **+ Add** to add a new endpoint to be load balanced.

7. On the **Add endpoint** pane, enter the following values for the new endpoint, then select **Add**.

    - Type: **External endpoint**
    - Name: `primary`
    - Fully-qualified domain name (FQDN) or IP: `fabmedical-[SUFFIX]-ingress.[AZURE-REGION].cloudapp.azure.com`
    - Location: Choose the same Azure Region as AKS.
    - Custom Header Settings: leave blank.
    - Add as disabled: leave unchecked.

    Be sure to replace the `[SUFFIX]` and `[AZURE-REGION]` placeholders.

    ![Add endpoint configuration pane with values entered.](media/tm-add-endpoint-primary.png "Add endpoint configuration")

8. Notice the list of **Endpoints** now shows the **primary** endpoint that was added.

9. On the **Traffic Manager profile** blade, select **Overview**.

10. On the **Overview** pane, copy the **DNS name** for the Traffic Manager profile.

    ![The Traffic Manager profile overview pane with the DNS name highlighted](media/tm-overview.png "fabmedical Traffic Manager profile DNS name")

11. Open a new web browser tab and navigate to the Traffic Manager profile **DNS name** that as just copied. You will receieve a 404 Not Found error. This is because the browser is not sending a host header that the ingress knows about.

      You could request the URL using the `curl` command line to see it function as expected. The following will return the JSON result for speakers.

      ```bash
      curl  -H "Host: fabmedical-[SUFFIX]-ingress.[AZURE-REGION].cloudapp.azure.com" http://fabmedical-[SUFFIX].trafficmanager.net/content-api/speakers
      ```

12. In Azure Cloud Shell open the ingress deployment you created in Step 17 of the last Task. We will add a host entry so that requests using the Traffic Manager hostname work without needing to specificy a host header that maps to the ingress hostname.

   ```bash
    code content.ingress.yml
   ```

    Use the following as the contents and update the `[SUFFIX]` and `[AZURE-REGION]` to match your ingress DNS name:

   ```yaml
   apiVersion: networking.k8s.io/v1beta1
   kind: Ingress
   metadata:
      name: content-ingress
      annotations:
         kubernetes.io/ingress.class: nginx
         nginx.ingress.kubernetes.io/rewrite-target: /$1
         nginx.ingress.kubernetes.io/use-regex: "true"
         nginx.ingress.kubernetes.io/ssl-redirect: "false"
         cert-manager.io/cluster-issuer: letsencrypt-prod
   spec:
      tls:
         - hosts:
         - fabmedical-[SUFFIX]-ingress.[AZURE-REGION].cloudapp.azure.com
         secretName: tls-secret
      rules:
         - host:fabmedical-[SUFFIX]-ingress.[AZURE-REGION].cloudapp.azure.com
         http:
            paths:
               - path: /(.*)
               backend:
                  serviceName: web
                  servicePort: 80
               - path: /content-api/(.*)
               backend:            
                  serviceName: api
                  servicePort: 3001
         - host: fabmedical-[SUFFIX].trafficmanager.net
         http:
            paths:
               - path: /(.*)
               backend:
                  serviceName: web
                  servicePort: 80
               - path: /content-api/(.*)
               backend:            
                  serviceName: api
                  servicePort: 3001
   ```
      > Note: you can [download this as a YAML file](lab-files/yaml-templates/ex5-task4-ingress-tm.yaml) and use the file as your template. Make sure to update the placeholders.


      Save the file and apply the changes to your AKS cluster by issuing the following command.

      ```bash
      kubectl apply -f content.ingress.yml
      ```

      Refresh the web browser which previoulsy received the 404 error and you should now see the website load as expected.

    ![The screenshot shows the Contoso Neuro website using the Traffic Manager profile DNS name](media/tm-endpoint-website.png "Traffic Manager show Contoso home page")

12. When setting up a multi-region hosted application in AKS, you will setup a secondary AKS in another Azure Region, then add its endpoint to its Traffic Manager profile to be load balanced.

    > **Note:** You can setup the secondary AKS and instance of the Contoso Neuro website on your own if you wish. The steps to set that up are the same as most of the steps you went through in this lab to setup the primary AKS and app instance.

## After the hands-on lab

**Duration**: 10 minutes

In this exercise, you will de-provision any Azure resources created in support of this lab.

1. Delete the Resource Groups in which you placed all your Azure resources.

   - From the Portal, navigate to the blade of your **Resource Group** and then select **Delete** in the command bar at the top.

   - Confirm the deletion by re-typing the resource group name and selecting Delete.

2. Delete the Service Principal created on Task 3: Create a Service Principal before the hands-on lab.

   ```bash
   az ad sp delete --id "Fabmedical-sp"
   ```

You should follow all steps provided _after_ attending the Hands-on lab.

[logo]: https://github.com/Microsoft/MCW-Template-Cloud-Workshop/raw/master/Media/ms-cloud-workshop.png
[portal]: https://portal.azure.com

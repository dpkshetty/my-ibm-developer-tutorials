---
# Added check date LCM backlog LA
check_date: "2025-10-01"
draft: false
ignore_prod: false
display_in_listing: true
title: "Deploy a 3-tier application on Red Hat OpenShift"
subtitle: "Use modern hybrid cloud tools and techniques to rapidly deploy a new multi-tier application in a flexible and secure way"
meta_title: "Deploy a 3-tier application on Red Hat OpenShift"
authors:
  - email: deepakcshetty@in.ibm.com
    name: Deepak C Shetty
  - email: andrew.laidlaw@uk.ibm.com
    name: Andrew Laidlaw
completed_date: '2022-07-08'
last_updated: '2022-07-08'
excerpt: "Use modern hybrid cloud tools and techniques to rapidly deploy a new multi-tier application in a flexible and secure way"
meta_description: "Use modern hybrid cloud tools and techniques to rapidly deploy a new multi-tier application in a flexible and secure way"
meta_keywords: "Multi-tier, 3-tier, application deployment,microservices, microservices based development, microservices based deployment, application modernization"
primary_tag: linux-on-ibm-power
tags:
  - linux
components:
  - ibm-power
  - redhat-openshift-ibm-cloud
related_links:
  - title: Sample source code for the database tier
    url: https://github.com/dpkshetty/qod-db
  - title: Sample source code for the API tier
    url: https://github.com/dpkshetty/qod-api
  - title: Sample source code for the web tier
    url: https://github.com/dpkshetty/qod-web
  - title: Quote of the Day database in GitHub
    url: https://github.com/dpkshetty/qod-db
also_found_in:
  - "learningpaths/exploring-openshift-powervs/"
---
## Introduction

In this tutorial, we will show you how to use the graphical console provided by Red Hat OpenShift Container Platform to rapidly deploy a 3-tier application in a hybrid cloud environment. Our application includes:

* A MariaDB database at the back end to hold our application data
* An application programming interface (API) tier based on Node.js code that connects to the database and provides some business logic
* A web facing front end also based on Node.js code that our users interact with

This tutorial is designed to be simple to follow as you **learn the concepts and terminology of the OpenShift Container Platform, and so only uses the graphical interface of OpenShift Container Platform (also known as the web console)**. This gives a visual representation of the application components and does not require the use of the command-line interfaces or to create configuration files using languages like YAML.

By using the graphical tools, you will be able to see how the different components of a hybrid cloud application are deployed and connected together. You can also experience how easy it is to connect separate microservices as the platform removes some of the complexity you would experience in a more traditional virtual machine (VM)-based environment. This includes the networking setup and routing required to connect components together, as well as the security benefits of disconnecting the database access credentials from the user.

The application we deploy is called “Quote of the Day” (QOD).

Once deployed in full, this application provides you with a simple webpage with the ability to get a daily quote which is different each day, or a random quote from our database of quotes.

The source code for the  following three tiers of the QOD application is hosted in GitHub:

* <a href="https://github.com/dpkshetty/qod-db" target="_blank" rel="noopener noreferrer">Database tier</a>
* <a href="https://github.com/dpkshetty/qod-api" target="_blank" rel="noopener noreferrer">API tier</a>
* <a href="https://github.com/dpkshetty/qod-web" target="_blank" rel="noopener noreferrer">Web tier</a>

After the deployment is complete, our 3-tier application looks as shown in the following figure.

![image](https://github.com/user-attachments/assets/c6b4887a-3daf-4b97-a932-5d0c9ecd160f)

## Prerequisites

Before you deploy this 3-tier application, make sure that the following prerequisites are fulfilled:

* Access to a Red Hat OpenShift Container Platform cluster. I am using **OpenShift version 4.10.xx on IBM Power Virtual Server**. The steps mentioned in this tutorial should work on any other OpenShift platform as well, although some of the GUI screens or dialog boxes might look slightly different.
* If you do not have access to a Red Hat OpenShift Container Platform cluster, you can access it through the Red Hat <a href="https://developers.redhat.com/developer-sandbox" target="_blank" rel="noopener noreferrer">developer sandbox</a>. Alternatively, you could deploy your own <a href="https://developer.ibm.com/learningpaths/getting-started-openshift-powervs/" target="_blank" rel="noopener noreferrer">OpenShift Container Platform cluster on IBM Power Virtual Server</a>.

## Estimated time

1 hour

## Steps

1. [Create a new project](#step1)
1. [Deploy the database (DB) tier](#step2)  
   2a. [Test DB tier (optional)](#step2a)
1. [Deploy the API tier](#step3)  
   3a. [Test the API service – It doesn’t work](#step3a)  
   3b. [Inject environment variables using a secret](#step3b)  
   3c. [Test the API service – It works!](#step3c)  
   3d. [Create a HTTP or HTTPS (as applicable) route for API test automation (optional)](#step3d)  
1. [Deploy the web tier](#step4)  
   4a. [Inject environment variables during pod creation](#step4a)

<a id='step1' />

## 1. Create a new project

1. Create a new OpenShift project where we will deploy our 3-tier application. In the **Administrator** view, click **Projects** and then click **Create Project**.
   
   ![image](https://github.com/user-attachments/assets/70776da1-381d-4d17-93cc-eeb4435f962d)  

1. In the Create Project dialog box, enter a name for your new project and click **Create**. In my case I am creating a new project, named **tutorial**.

   ![image](https://github.com/user-attachments/assets/ad317778-5382-4f15-b9aa-f03e2161861d)

<a id='step2' />

## 2. Deploy the database (DB) tier

In this section, we will deploy the database tier to host a MariaDB database named qod and it has tables such as quotes, genres, and authors which will be pre-populated with quotes, the different genres of the quotes, and the author of the quotes respectively.

1. Switch to **Developer** persona, make sure your project is selected (‘tutorial’ in my case) and click **+Add**.

   ![image](https://github.com/user-attachments/assets/1dd32401-e74e-41e0-9681-4134b8921f12)

1. Click **Import from Git**.

   ![image](https://github.com/user-attachments/assets/1a22382d-2a96-4362-b5cf-8912fcdac984)

   **Note**: On older versions of OpenShift, there may be a separate **From Dockerfile** option, which should be chosen instead. 

1. In the Import from Git dialog box, fill in the following fields with the given values:

   * **Git Repo URL**: <a href="https://github.com/dpkshetty/qod-db" target="_blank" rel="noopener noreferrer">https://github.com/dpkshetty/qod-db</a>  
       *(Press Tab and wait for it to show ‘Validated’)*  
      OpenShift will examine the Git repo and auto-detect the import strategy as Dockerfile, due to the <a href="https://github.com/dpkshetty/qod-db/blob/master/Dockerfile" target="_blank" rel="noopener noreferrer">Dockerfile</a> present in the qod-db repository.
   * **Application Name**: QOD
   * **Name**: qod-db (This name is used for all the resources that OpenShift creates, including the DNS entry for accessing the DB microservice)
   * Keep **Deployment** (the default) selected.
   * **Target port**: 3306 (refer to the <a href="https://github.com/dpkshetty/qod-db/blob/master/README.md" target="_blank" rel="noopener noreferrer">README.md</a> file to find the port on which the DB service is designed to listen for connections).
   * Clear the **Create a route to the Application** checkbox.  
   **Note**: We don’t need to expose the DB service to the outside world - hence a route to the DB service is not needed. It will only be accessed from inside the cluster and not having an external route makes the DB service more secure from external attacks too!).
   * Click **Create**.

   ![image](https://github.com/user-attachments/assets/2db63613-1490-4854-a2d0-4d8c232e334a)
   ![image](https://github.com/user-attachments/assets/9c7172b0-945c-4157-bc9f-1cc99e63b1ef)
   ![image](https://github.com/user-attachments/assets/ca10ed41-db9d-4d1b-8d4f-62ed2687fcd5)

   _**What happens next** is that OpenShift will fetch the source code from the GitHub repository, use the Dockerfile to create a Docker image, save that image into the internal OpenShift image registry, and create a pod (also known as application) from that Docker image, which will be our DB microservice application (our back-end tier). The application will instantiate a MariaDB instance, and create a database named **qod** which will have three tables named quotes, genres, and authors, all pre-populated with different quotes, genres, and authors respectively._

1. You will be taken back to the Topology page. Click **D qod-db** that represents the deployment object. Wait for the build process to finish and the pod to be in the **Running** state. This pod is our DB microservice available on port 3306 (as seen from the service information) and the absence of a route ensures that the service cannot be accessed from outside the cluster.
   
   **Note**: If you click the icon at the lower left side of the large circle while the build is running, you can see the logs of the build process as it is run. 

   ![image](https://github.com/user-attachments/assets/fc3106c7-d3a4-48f3-a2c8-8f76271c32dc)

**Congratulations!** You have successfully deployed the DB tier, having a DB named qod and pre-populated with quotes, authors, and genre tables.

<a id='step2a' />

### 2a. Test DB tier (optional)

For those who want to go one step deeper and check if the database and the different tables were created successfully, we can get inside the Pod and use the MySQL CLI to inspect. If you check the <a href="https://github.com/dpkshetty/qod-db/blob/master/Dockerfile#:~:text=ENV%20MYSQL_USER%3Duser,ENV%20MYSQL_DATABASE%3Dqod" target="_blank" rel="noopener noreferrer">Dockerfile</a>, the DB credentials are hard-coded in there, and so we will use those to access the DB and its tables.

1. Click **View Logs** to go to the Pod view, then click **Terminal** to get inside the pod and run the MySQL CLI as shown in the following screen captures.
  
   ![image](https://github.com/user-attachments/assets/78f4e1ed-52be-4908-bf7a-77ab27ea9e26)
   ![image](https://github.com/user-attachments/assets/5f19f067-dd5f-4cd7-87e4-9c79b944df52)
   ![image](https://github.com/user-attachments/assets/2789358e-0c5d-4cdf-9679-090c6e304001)

   The MySQL CLI (shown in the above screen capture) uses username as user and password as *pass* (taken from the Dockerfile) and the hostname as *qod-db* (which is same as the name that was used when deploying the DB service).

   ![image](https://github.com/user-attachments/assets/ab07ecaf-451d-4502-b3e5-91dcb921820f)
   ![image](https://github.com/user-attachments/assets/48ec6feb-b30a-4f27-a115-202210fba7ff)

1. Notice that authors, genres, and quotes tables are pre-created.
1. If required, run any SQL command to query these tables as shown in the following screen capture.

   ![image](https://github.com/user-attachments/assets/02ee1bc6-3b27-4bc5-b7ed-544d859e0152)

   Notice that the SQL query command  lists around 500 rows, each row having a different quote.

   ![image](https://github.com/user-attachments/assets/75d9108a-6f16-42c7-a52a-0c37a3dbb4c2)

<a id='step3' />

## 3. Deploy the API tier

The API tier provides the business logic layer (in the simplest sense) for the application. It receives the request from the user (typically from the front-end web tier), processes the request by running SQL queries against the DB, and provides the response back to the user through the web interface. In the case of our QOD application, the API tier is designed to process the requests for daily and random quotes by forming the right SQL query and retrieving the quotes from the database using the qod-db microservice.

1. Click **+Add** and then click **Import from Git**.
   
   ![image](https://github.com/user-attachments/assets/be70e547-2196-402c-b675-1301ce8e4856)

1. In the Import from Git dialog box, enter the following values in the required fields:

   * **Git Repo URL**: <a href="https://github.com/dpkshetty/qod-api" target="_blank" rel="noopener noreferrer">https://github.com/dpkshetty/qod-api</a>  
   *(Press Tab and wait for it to show **Validated**)*  
   OpenShift will examine the Git repo and autodetect the import strategy as Dockerfile, due to the <a href="https://github.com/dpkshetty/qod-api/blob/master/Dockerfile" target="_blank" rel="noopener noreferrer">Dockerfile</a> present in the qod-api repo.

     **Note**: Alternatively, You can also choose your own build strategy, including the option to build an image directly from the source code. This uses the Red Hat Source to Image capability and removes the requirement for developers to write their own Dockerfiles.
   * **Application Name**: select QOD from the existing application group using the drop-down list.
   * **Name**: qod-api (This name is used for all the resources that OpenShift creates, including the DNS entry for accessing the API microservice).
   * Keep **Deployment** (the default) selected.
   * **Target port**: 8080 (The <a href="https://github.com/dpkshetty/qod-api/blob/master/README.md" target="_blank" rel="noopener noreferrer">README.md</a> file states the port at which the API service is designed to run).
   * Clear the **Create a route to the Application** checkbox.

     **NOTE**: We don’t need to expose the API service to the outside world, and hence a route to the API service is not needed. It will only be accessed from inside the cluster and not having an external route makes the API service more secure from external attacks too!
1. Click **Create**.

   ![image](https://github.com/user-attachments/assets/be220707-f0d9-4090-8f4e-b909803ee1d1)
   ![image](https://github.com/user-attachments/assets/869b0854-f62f-4b89-9982-09ca81f2bba4)
   ![image](https://github.com/user-attachments/assets/933889c3-5fab-4891-ae36-870adf15bd5c)

   _**What happens next** is that OpenShift will fetch the source code from the Git repo, use the Dockerfile to create a Docker image, save that image into OpenShift internal image registry, and create a pod (also known as application) from that Docker image, which will be our API microservice application, our middle tier._

1. You will be taken back to the Topology page. Click **D qod-api** which represents the deployment object. Wait for the build process to finish and the pod to be in **Running** state. This pod is our API microservice available on port 8080 (as seen from the service information) and the absence of a route ensures the service cannot be accessed from outside the cluster.
   
   ![image](https://github.com/user-attachments/assets/8c29dcd9-d3ae-49be-a8eb-5506643a0988)

<a id='step3a' />

### 3a. Test the API service – It doesn’t work

1. Let’s quickly check if the API service works as expected. Click the qod-api pod to enter the Pod view and then click **Terminal**. This will log you inside the API service pod and present you with a Linux shell.

   ![image](https://github.com/user-attachments/assets/dd9ec874-f95e-48cd-962e-2810b4131d2d)
   ![image](https://github.com/user-attachments/assets/7623b659-8580-4d10-88d6-fa8671fd870a)

2. The API service exposes many REST API endpoints, one of them being the `/daily` endpoint. If interested, refer to the <a href="https://github.com/dpkshetty/qod-api/blob/master/app.js" target="_blank" rel="noopener noreferrer">code</a>. This endpoint returns back the quote of the day based on today’s date. We know that the API service is running on port 8080. Use the `curl` utility to test if the `/daily` endpoint works as expected. Type the following command in the terminal and check the response. It throws an error!

   `curl http://localhost:8080/daily`

   ![image](https://github.com/user-attachments/assets/da29222e-36a1-4654-8748-edf14f2674f5)

   The error is expected because the API service needs to communicate with the DB service in order to process incoming requests. We haven’t provided any configuration information to the API service instructing it on how to connect to the DB service. If you look at the API service <a href="https://github.com/dpkshetty/qod-api/blob/master/app.js#:~:text=host%20%20%20%20%20%3A%20process,.DB_PASS%2C" target="_blank" rel="noopener noreferrer">app.js</a> code, you will see that it needs the following environment variables to be provided in order for it to connect to the DB service. The same is mentioned in the API service <a href="https://github.com/dpkshetty/qod-api/blob/master/README.md" target="_blank" rel="noopener noreferrer">README</a> file too!

   * DB_HOST
   * DB_USER
   * DB_PASS

<a id='step3b' />

### 3b. Inject environment variables using a secret

OpenShift provides a neat way to inject environment variables into a Pod, using the Secret functionality. Let’s create a secret and inject it into the API pod.

1. Click **Secrets** on the left navigation menu and click **Create**. Then click **Key/Value secret**.

   ![image](https://github.com/user-attachments/assets/d0abb1f6-a88b-49fe-afec-4f7c3d5b1f73)

   ![image](https://github.com/user-attachments/assets/ba23d6e1-4949-4e22-bca3-42e3f01628c4)

1. On the create Key/Value secret page, enter the following values for the required fields:
   * **Secret Name**: qod-db-credentials
   * **Key**: DB_HOST
   * **Value**: qod-db
   1.	Click **+Add key/value** to add a new entry and enter the following values:
        * **Key**: DB_USER
        * **Value**: user
   1.	Click **+Add key/value** to add a new entry and enter the following values:
        * **Key**: DB_PASS
        * **Value**: pass
   1.	Click **Create**.

   **Note**: These are the environment variables needed by the API service to access the DB service as documented in the <a href="https://github.com/dpkshetty/qod-api/blob/master/README.md" target="_blank" rel="noopener noreferrer">README.md</a> file.

   ![image](https://github.com/user-attachments/assets/0563c3e2-f9f2-41f8-a6f4-5f72604d20ac)

1. Click **Add Secret to workload**. On the Add secret to workload page, select the **qod-api** deployment option  from the drop-down List, select the **Environment variables** option and click **Save**.

   ![image](https://github.com/user-attachments/assets/a4e422a2-970c-4157-956f-87ce0d502e23)
   ![image](https://github.com/user-attachments/assets/d4ced668-90ec-4d85-9b90-7c49b30f72f9)

   _**What happens next** is that OpenShift will restart the **qod-api** pod with these environment variables injected, and this allows the API service to communicate with the DB service._
1. Click **Topology** on the left navigation pane, and then click **D qod-api** to open the **Resources** tab and wait for new pod to be in the **Running** state.

   ![image](https://github.com/user-attachments/assets/7d394e93-155b-49af-b083-c1f292bf0ea7)

<a id='step3c' />

### 3c. Test the API service – It works!

1. Click the **qod-api** pod to enter the pod view and then click **Terminal**. This will log you inside the API service pod and opens a Linux shell.

   ![image](https://github.com/user-attachments/assets/8dbcb332-9cdf-4595-9c39-c6d7aa3c60c8)
   ![image](https://github.com/user-attachments/assets/1455d537-0222-40a7-b619-6487a13b3535)

1. Run the `curl` command, and this time, it should run successfully and return us the daily quote, along with its quote ID, author, and genre information.
   
   `curl http://localhost:8080/daily`

   ![image](https://github.com/user-attachments/assets/871ad08a-2dd1-45f6-9e87-573d7253eca7)

   **Congratulations!** You have successfully deployed the API tier and linked it with the DB tier using secrets to pass the DB credentials to the API tier. The output of the API service is in the JSON format which is not very user-friendly. This is where the web tier (covered later in the tutorial) comes into play, which consumes the API service’s output and presents it in an intuitive user-friendly way.

<a id='step3d' />

### 3d. Create a HTTP or HTTPS (as applicable) route for API test automation (optional)

In the previous steps we accessed the \`*/daily*` REST endpoint by getting inside the pod. While this is fine for a quick check, it is not automation friendly as it involves many GUI steps. In a real world scenario, we would want to test the API layer using automated test scripts which can then be part of the DevOps build, test, and release workflow.

For testing purposes, we can create a HTTP or HTTPS (as applicable for your cluster) route for the API service such that its accessible external to the cluster. We can then use the route URL as a target in our API test automation scripts.

1. To create a route for API service, switch to the **Administrator** view and click **Networking –> Routes**. Then click **Create Route**.

   ![image](https://github.com/user-attachments/assets/bd1f728c-b419-43ae-942c-5c21e412007d)

1. On the Create Route page, fill in the following values in the required fields:
   
   * **Name**: qod-api-route
   * **Service**: qod-api
   * **Target port**: 8080

   **Note**: My cluster mandates to use the HTTPS secure routes. You can opt to skip the following steps if this is not applicable to your cluster. You will be creating a HTTP (unsecure) route.

    ![image](https://github.com/user-attachments/assets/97f3b585-beee-4a0f-8cd8-6df422b06508)

   * Select the **Secure Route** checkbox (optional).
   * **Specify the values for the following options**:
     * **TLS Termination**: Edge (optional)
     * **Insecure Traffic**: None (optional)
   * Retain the default values for the remaining fields and click **Create**.

    ![image](https://github.com/user-attachments/assets/3beb0de6-4e7a-495d-add5-b7b473ff4610)

1. On the Route details page, click the location URL (that begins with https://... in my case). A new browser tab opens and displays the API server version.

    ![image](https://github.com/user-attachments/assets/f7c5efed-13d0-4dec-a0fb-fe515c5699cd)
    ![image](https://github.com/user-attachments/assets/b2d282c3-1f09-4111-89bd-d2e80d08daea)

1. Similarly, you can edit the URL and replace /version with /daily or /random to get daily and random quotes respectively.

    ![image](https://github.com/user-attachments/assets/7b04a8c7-c0e7-443b-9ad6-fe10b45ce597)

    ![image](https://github.com/user-attachments/assets/1e23428a-4944-419e-82be-c7680f2b192d)

1. Optionally, retrieve a particular quote or genre by its ID too.

    ![image](https://github.com/user-attachments/assets/33c449e5-92ab-4b08-8a97-2329d337a7fd)

    ![image](https://github.com/user-attachments/assets/f425a26e-8d0c-498c-aa86-a3462fa1e7f4)

   **Congratulations!** You have successfully added a route to the API service and made its REST endpoints accessible from outside the cluster. This method can be used to run any API-level test automation after which the route can be deleted to secure the API service from outside attacks.

   In case you need to remove the route, in the **Administrator** view, click **Routes** and search for **qod-api-route** (if needed).
 
    ![image](https://github.com/user-attachments/assets/2ce60cbe-8e34-49d2-8aa1-38dca73d617b)

1. Click the vertical ellipsis icon (with three dots), click **Delete Route**, and then click **Delete** to delete the route.

    ![image](https://github.com/user-attachments/assets/b01b8ec5-79d5-4009-9bdc-a45f2a6c7443)

    ![image](https://github.com/user-attachments/assets/bbb53f92-19f8-4cc5-8bf0-484203899192)

    ![image](https://github.com/user-attachments/assets/8a7d3d76-3066-434b-9600-5930fc8eca34)

<a id='step4' />

## 4. Deploy the web tier

In this section, we will deploy our front-end web tier. The web tier will have an external route because it is the service that the users interact with. Depending on the user request, it either displays the daily quote or a random quote.

1. Switch to the **Developer** persona, make sure your project is selected (**tutorial** in my case), and click **+Add**.

    ![image](https://github.com/user-attachments/assets/0e29b785-189e-4517-a4e3-0b504a9cff5e)

1. Click **Import from Git**.

    ![image](https://github.com/user-attachments/assets/3afe5c4c-b471-4f44-8699-7954fcff6fb8)

1. On the Import from Git page, fill the required values in the following fields:
   * **Git Repo URL**: <a href="https://github.com/dpkshetty/qod-web" target="_blank" rel="noopener noreferrer">https://github.com/dpkshetty/qod-web</a>  
       (Press **Tab** and wait for it to show **Validated**)  
      OpenShift will examine the Git repo and auto-detect the import strategy as <a href="https://github.com/dpkshetty/qod-web/blob/master/Dockerfile" target="_blank" rel="noopener noreferrer">Dockerfile</a>, because the Dockerfile is present in the qod-db repository.
   * **Application**: QOD
   * **Name**: qod-web (this name is used for all the resources that OpenShift creates, including the DNS entry for accessing the web microservice)
   * Keep Deployment (the default) selected
   * **Target port**: 8080 (The <a href="https://github.com/dpkshetty/qod-web/blob/master/README.md" target="_blank" rel="noopener noreferrer">README.md</a> file states the port at which the web service is designed to run)
   * Select the **Create a route to the Application** checkbox.
  
     **Note**: My cluster mandates HTTPS routes. If that is not the case for your cluster, you can skip the following steps used for creating a HTTPS route.
   * Select the **Secure Route** checkbox, and enter the following values for the other fields:
      * **TLS Termination**: Edge
      * **Insecure Traffic**: None
   * Retain the default values for the remaining fields.

     **Note**: **Do not** click **Create** yet! Continue with the next step.

    ![image](https://github.com/user-attachments/assets/d207cd85-4387-4bfb-9573-004f94fa8243)

    ![image](https://github.com/user-attachments/assets/fd674925-8346-4257-9d18-4598546b514a)

    ![image](https://github.com/user-attachments/assets/839f2068-deb6-49b1-a77b-e2e1f4008704)

    ![image](https://github.com/user-attachments/assets/b7f3f923-2279-40b6-98aa-54fb3cc7fd1e)

<a id='step4a' />

### 4a. Inject environment variables during pod creation

Like we did for the API tier, the web tier needs to be linked with the API tier for it to work successfully. From the <a href="https://github.com/dpkshetty/qod-web/blob/master/README.md" target="_blank" rel="noopener noreferrer">README.md</a> file, we can see that we need to define the `QOD_API_URL` environment variable. OpenShift provide a way to specify the environment variables during pod creation.

1. Scroll down to the bottom of the form and click **Deployment**. which will bring up the environment variables option.
   
   ![image](https://github.com/user-attachments/assets/172b0831-f37b-46db-91dc-645415d2ec00)

   ![image](https://github.com/user-attachments/assets/4f644a71-d67f-4c9f-abc0-359ec69f3642)

1. On the Deployment page, set the following values for the environment variable:
   * **Name**: QOD_API_URL
   * **Value**: `http://qod-api:8080` (do not use a trailing slash at the end)
  
   Then, click **Create**.

   ![image](https://github.com/user-attachments/assets/8c489927-c9c9-4c02-a13d-49194835b27e)

   _**What happens next** is that OpenShift will fetch the source code from the Git repository, use the Dockerfile to create a Docker image, save that image into OpenShift internal image registry, and create a pod (also known as application) from that Docker image, which will be our web  microservice application (our front-end tier). The HTTPS (or HTTP, as applicable) URL will serve our web application for users._

1. On the Topology page, click **D qod-web** representing the deployment object. Wait for the build to finish and the pod to be in **Running** state. This pod is our web microservice available on port 8080 (as seen from the service information) and the external HTTPS (or HTTP, as applicable) route has been created.

   ![image](https://github.com/user-attachments/assets/38a7a394-47ff-42c7-bf19-3b93bf8564f8)

   ![image](https://github.com/user-attachments/assets/23eccab6-c6ea-4de9-8a8d-2ecb03947bab)

1. Click the HTTPS (or HTTP, as applicable for your cluster) URL and notice that your web application is running successfully and displaying the quote of the day.

   ![image](https://github.com/user-attachments/assets/263dfd2a-ad40-4125-9ba4-56c645782c64)

   ![image](https://github.com/user-attachments/assets/8cf2460d-7fbe-4326-8901-792fb34815d2)

1. Click **Random Quote** to display a random quote picked from the DB each time.

   ![image](https://github.com/user-attachments/assets/8f631cf5-db6d-4db0-af30-c531b42f2f25)

   ![image](https://github.com/user-attachments/assets/c70450f0-deb8-4aaf-b047-b291322ffd39)

   ![image](https://github.com/user-attachments/assets/ffe02ca0-85ba-4647-86e6-fb8f719667ae)

   This proves that the `QOD_API_URL` environment variable we initialized during the pod creation process using the Deployment field (under advanced options) worked and the web tier is able to communicate with the API tier successfully!

   **Congratulations!** You have successfully deployed the front-end web tier of your application and linked it with the API tier using environment variables and exposed an external route/URL for users to access your application.

1. **(Optional)** OpenShift provides visual connectors in the Topology view to depict how the different services connect with each other. In the Topology view, hover over each service until you see an arrow pop up and drag it to connect to the other service. As shown in the following screen capture, it helps to better visualize the flow of traffic across the different services making up your application.

   ![image](https://github.com/user-attachments/assets/86d6ab70-1726-4166-9e11-34e1babbc06d)

## Summary

In this tutorial, you have learnt how to deploy a simple 3-tier application in Red Hat OpenShift Container Platform using stand-alone microservices for each of the tiers. You have also learnt how to pass the credentials of a microservice to another microservice using secrets, and how to link the microservices using environment variables. You also learnt how to secure the 3-tier application by exposing only the web tier to the outside world while keep the API and database tiers secure. Most importantly, you have seen how quickly you can get a new application deployed within the OpenShift environment, without the need to set up new virtual machines, and networking or development environments, and thus speeding up the development duration and reducing the administrative burden.

There might be circumstances where you want to automate the build and deploy of your individual application tiers in response to a Github code change to achieve DevOps style CI/CD automation for your application. In such cases, refer to the <a href="https://developer.ibm.com/tutorials/continuous-deployment-s2i-and-webhooks/" target="_blank" rel="noopener noreferrer">Enable continuous deployment using Red Hat OpenShift S2I and GitHub webhooks</a> tutorial to learn how to use GitHub webhooks with OpenShift to create an automated workflow. In case your OpenShift cluster is behind an enterprise firewall, you can refer to the <a href="https://developer.ibm.com/learningpaths/exploring-openshift-powervs/deliver-your-webhooks-without-worrying-about-firewalls/" target="_blank" rel="noopener noreferrer">Deliver webhooks without worrying about firewalls</a> tutorial to learn how to get GitHub webhooks work through the firewall.

## Take the next step

Join the <a href="https://community.ibm.com/community/user/powerdeveloper/home" target="_blank" rel="noopener noreferrer">Power Developer eXchange Community (PDeX)</a>. PDeX is a place for anyone interested in developing open source apps on IBM Power. Whether you're new to Power or a seasoned expert, we invite you to join and begin exchanging ideas, sharing experiences, and collaborating with other members today!

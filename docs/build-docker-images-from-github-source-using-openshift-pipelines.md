---
# Added check date LCM backlog LA
check_date: "2025-05-19"
draft: false
ignore_prod: false
display_in_listing: true
title: 'Build and publish Docker images from a GitHub source using Red Hat OpenShift Pipelines'
subtitle: Achieve DevOps style continuous deployment for your Docker images
meta_title: 'Build and publish Docker images from a GitHub source using Red Hat OpenShift Pipelines'
authors:
  - name: "Deepak C Shetty"
    email: deepakcshetty@in.ibm.com
completed_date: '2022-05-19'
last_updated: '2022-05-19'
excerpt: Achieve DevOps style continuous deployment for your Docker images
meta_description: Achieve DevOps style continuous deployment for your Docker images
meta_keywords: OpenShift Pipelines, tekton, continuous deployment, DevOps
primary_tag: linux-on-ibm-power
tags:
  - infrastructure
  - linux
  - devops
  - ci-cd
components:
  - redhat-openshift-ibm-cloud
  - ibm-power
  - docker
related_links:
  - title: Introducing OpenShift Pipelines
    url: https://cloud.redhat.com/blog/introducing-openshift-pipelines
  - title: Tekton
    url: https://tekton.dev/
  - title: Red Hat OpenShift documentation
    url: https://docs.openshift.com/container-platform/4.8/cicd/pipelines/working-with-pipelines-using-the-developer-perspective.html
  - title: Sample GitHub repository
    url: https://github.com/dpkshetty/pipeline-s2i-pyflask-demo
also_found_in:
  - "learningpaths/exploring-openshift-powervs/"
---
## Introduction

A pipeline is a sequence of steps to represent a software development workflow (build, test, deploy) also known as continuous integration / continuous deployment (CI/CD). DevOps engineers are always looking to automate this workflow to minimize human error, improve time to deliver software, and produce consistent software artifacts.

Red Hat OpenShift Pipelines is a CI/CD solution based on the open source <a href="https://tekton.dev/" target="_blank" rel="noopener noreferrer">Tekton</a> project. The main objective of Tekton is to enable  DevOps teams to quickly create pipelines for activities involving simple, repeatable steps. A unique characteristic of Tekton that differentiates it from the previous CI/CD solutions is that Tekton procedure runs within a container that is specifically created just for that task. This provides a degree of isolation that supports predictable and repeatable task execution and ensures that development teams do not have to manage a shared build server instance.

Most OpenShift pipelines related blogs, articles, and how-to guides use complicated YAML files and a command-line interface (CLI) to create, deploy, and run pipelines, which isn’t easy for many to adapt to and follow!

This tutorial takes a different approach and uses the OpenShift GUI / console. **There isn’t a single YAML or CLI command used in this tutorial**, yet it shows you how to create a new pipeline from scratch, set up tasks to build from GitHub and deploy a Docker image to quay.io (a popular image registry) and how to achieve continuous delivery and deployment of Docker images by automating the whole process using OpenShift Pipeline triggers and GitHub webhooks.

This tutorial would benefit users interested in understanding OpenShift Pipelines without getting into complicated YAMLs and CLIs, users new to the pipelines concept, or users looking to get a quick understanding of how pipelines work.

## Prerequisites

Before you build and publish Docker images from a GitHub source, make sure that the following prerequisites are fulfilled:

* Access to an OpenShift cluster. OpenShift version 4.7.x or later should be installed as the pipelines feature was introduced in this version. **I am using OpenShift version 4.8.xx on IBM Power Virtual Server**. The steps mentioned in this tutorial should work on any OpenShift platform as pipelines functionality is similar irrespective of the underlying hardware architecture.
* An OpenShift cluster configured with at least one storage class (to supply storage to the pipeline tasks).
* Familiarity with basic Git (git clone, edit code in Git web UI, and commit) operations.
* Familiarity with quay.io (registry for storing and building container images).

## Estimated time

It would take around one hour to build and publish Docker images from a GitHub source using Red Hat OpenShift Pipelines.

## Steps

Perform the following steps to build and publish Docker images from a GitHub source using Red Hat OpenShift Pipelines:

1. [Install OpenShift Pipelines operator (optional).](#sec1)
1. [Clone Git repository.](#sec2)
1. [Create a new quay.io repository to publish Docker image.](#sec3)
1. [Create a simple pipeline to build and publish Docker image from the GitHub source code.](#sec4)  
   4a. [Create a git-clone task.](#sec4a)  
   4b. [Create s2i-python task.](#sec4b)
1. [Run the pipeline.](#sec5)  
   5a. [Fix the secret bug (optional).](#sec5a)  
   5b. [Execute the pipeline.](#sec5b)
1. [Verify Docker image creation in quay.io.](#sec6)
1. [Validate the Docker image created in quay.io.](#sec7)
1. [Automate Docker image build using OpenShift Pipeline triggers and GitHub webhooks.](#sec8)  
   8a. [Set up pipeline triggers.](#sec8a)  
   8b. [Create a event listener HTTPS route URL (optional).](#sec8b)  
   8c. [Set up GitHub webhooks.](#sec8c)  
1. [Verify if a GitHub code change creates a new Docker image.](#sec9)
1. [Validate if the new version of the Docker image works.](#sec10)

<div id="sec1"></div>

### 1. Install OpenShift Pipelines operator (optional)

*(Skip this step if its already installed in your OpenShift cluster)*

1. Make sure you are in the **Administrator** persona.

   ![image](https://github.com/user-attachments/assets/29b8b992-9f5f-4ed6-8e5e-9ac16bf8dbef)  
   
1. Navigate to **OperatorHub**,  search for **pipeline**, and click the **Red Hat OpenShift Pipelines** tile.

   ![image](https://github.com/user-attachments/assets/2b4a6592-00b1-406c-a914-7195c26c102b)

1. On the page that shows the different installation options, retain the default values for this tutorial and click **Install**.

   ![image](https://github.com/user-attachments/assets/09179cea-2a8e-4e0a-bb20-c04111b15e9a)

1. The Install Operator page helps you with options to specify the update channel to subscribe to, the project or namespace the operator will be visible, and so on. For the example used in this tutorial, retain the default values and click **Install**.

   ![image](https://github.com/user-attachments/assets/4ed6dbd4-8399-4a99-a501-9470cfb2fa04)

1. Wait for the operator to get installed, and this might take a few minutes.

   ![image](https://github.com/user-attachments/assets/2012b745-aa24-4a8a-a4d5-cb6593dec36f)

1. Verify the installation by clicking the **Installed Operators** tab. Pipeline Operator should be listed there.

   ![image](https://github.com/user-attachments/assets/34dbe364-ee89-48d0-8cd2-21212e852ac3)
   
1. Notice that the **Pipelines** menu is now displayed.

   ![image](https://github.com/user-attachments/assets/05893238-6f15-4455-adc4-26cb0bc038e2)

1. Switch to **Developer** persona and ensure that the **Pipelines** menu is available there as well.

   ![image](https://github.com/user-attachments/assets/209a421e-0ad5-4dce-a249-a89106e8e759)

**Congratulations!** You have successfully installed Pipelines Operator and OpenShift Pipelines functionality is now available in your cluster.

<div id="sec2"></div>

### 2.	Clone Git repository

1. Clone the following Git repository as we will be using this simple Pyflask code in the GitHub repository to create a Docker image. You need to clone this as you require ownership permissions to edit code and create webhook (covered later in this tutorial).

   GitHub repository to fork/clone:
<a href="https://github.com/ocp-power-demos/s2i-pyflask-demo" target="_blank" rel="noopener noreferrer">https://github.com/ocp-power-demos/s2i-pyflask-demo</a>
1. After cloning, notice that your Git repository URL looks as follows:  
`https://github.com/<your_github_username>/s2i-pyflask-demo`  

In this tutorial, I am using a similar GitHub repository of mine located at:
<a href="https://github.com/dpkshetty/pipeline-s2i-pyflask-demo" target="_blank" rel="noopener noreferrer">https://github.com/dpkshetty/pipeline-s2i-pyflask-demo</a>

<a id="sec3"/>

### 3.	Create a new quay.io repository to publish Docker image

1. Login to <a href="https://quay.io/" target="_blank" rel="noopener noreferrer">https://quay.io/</a> (create a new login if needed).  
My login ID is **dpkshetty**.
1. Click the **Repositories** tab and then click **Create New Repository** to create a new repository to host the Docker image we will be creating in this tutorial.

   ![image](https://github.com/user-attachments/assets/9c1c0b42-053e-4694-9b2a-d002741d57ec)
   
1. In the resulting page, enter a name for your repository (I am using **demos**) and click **Public** to make it visible and accessible to others.

   ![image](https://github.com/user-attachments/assets/57602d31-ace8-421c-b845-182d11fa2957)
   
1. Scroll down, retain the default values, and click **Create Public Repository**.

   ![image](https://github.com/user-attachments/assets/5385e1f2-2ee1-4469-b053-19094dd468cd)
   
1. Notice that your new repository (for example **demos** in my case) is listed under **Repositories**.

   ![image](https://github.com/user-attachments/assets/059bbc60-d7e4-47d9-aba1-393793bcd153)

   Base URL for my repository (to pull or push Docker images) is: `quay.io/dpkshetty/demos`

   In your case it will be `quay.io/<your_username>/demos`

   In order to be able to push content to this repository, we need to create user credentials with the right permissions. In quay.io, this can be achieved by creating a new robot account. Let’s create one.

1. Click your username and then click **Account Settings**.

   ![image](https://github.com/user-attachments/assets/ac1cc790-fdda-4a8d-903c-f8696a023368)
   
1. Click the **Robot Accounts** icon, and then click **Create Robot Account**.

   ![image](https://github.com/user-attachments/assets/88352203-a25e-4425-8d9a-f2376a0b548e)
   
1. In the resulting form, enter a name (**demos** in my case) for your new robot account and click **Create robot account**.

   ![image](https://github.com/user-attachments/assets/2fe2b8a8-d19d-467a-8954-3a9a9baddf06)

1. In the next form, select the **demos** repository, select the **Write** permission, and click **Add Permissions**. By doing this we are allowing write (hence push) privileges to this robot account.

   ![image](https://github.com/user-attachments/assets/fc496887-fa57-45af-b6ec-171bdc064f31)

1. On the Robot Accounts page, notice that your new robot account is created successfully.

   ![image](https://github.com/user-attachments/assets/9c336a91-2957-4195-a250-4d1900e6b707)
   
**Congratulations!** You have successfully created a new quay.io repository and a new robot account with write (hence push) privileges. We will be using this repository and the associated robot account to publish our Docker image (later in this tutorial).

<div id="sec4"></div>

### 4.	Create a simple pipeline to build and publish Docker image from the GitHub source code

1. Switch to the **Developer** persona and create a new project (named **tutorial** in my case).

   ![image](https://github.com/user-attachments/assets/810e15ff-1410-4d5d-8661-e3a873cdfaaf)
   ![image](https://github.com/user-attachments/assets/49651f94-c835-40ae-a0d0-64a4cd6ae284)

1. Click **Pipelines** and then click **Create Pipeline**.

   ![image](https://github.com/user-attachments/assets/27add913-87bb-408d-92e5-e8fa049976ec)

1. On the Pipeline builder view that enables you to  create a new pipeline, enter a name to the pipeline (**create-pyflask-image** in my case).

   ![image](https://github.com/user-attachments/assets/eeb9c89c-a107-418f-8a01-da3bfd45252e)

<div id="sec4a"></div>

### 4a. Create a git-clone task

1.	From the **Select Task** drop-down list, select **git-clone**.

    ![image](https://github.com/user-attachments/assets/0a574096-cbc7-4505-bb30-c7584a251a79)
  	
1. Click the **git-clone** task to view its properties pane on the right side of the console.

    ![image](https://github.com/user-attachments/assets/f8625066-6f39-4d2b-b31b-413922dc7593)
    
1. Update the fields in the git-clone properties pane with the following values:

   * **Display name**: git-clone (the default)
   * **url**: `<`Enter your cloned GitHub repository URL (see step 2 in the [Clone Git repository](#sec2) section`>`

     I will be using
     <a href="https://github.com/dpkshetty/pipeline-s2i-pyflask-demo" target="_blank" rel="noopener noreferrer">https://github.com/dpkshetty/pipeline-s2i-pyflask-demo</a>
   * **revision**: main

    ![image](https://github.com/user-attachments/assets/a09b5e84-5d6a-47d5-ae22-72b18d4ebcb6)
  
1. Scroll down until you see the **Workspaces** section. Pipeline needs a workspace (storage area), but we don’t have any created yet. Thus, notice that the Select workspace field is disabled, and that’s expected!

   ![image](https://github.com/user-attachments/assets/7fa540a9-7568-4165-9a88-1cb99c2f18e9)

   Note that a pipeline has multiple tasks, and it needs a shared or a common storage to pass data between them. For example, the **git-clone** task will copy the source code which needs to be accessed by the next task (**s2i-python task**, covered further in the tutorial) which will build the Docker image. A workspace provides that common storage between tasks.
1. To create a workspace, go back to the Pipeline builder view/page (the middle pane in the browser), scroll down until you see the **Workspaces** section, click **Add workspace** , and enter a name to the workspace (**my-workspace** in my case).

   ![image](https://github.com/user-attachments/assets/00883178-ed85-4362-8d20-6673416b7d56)
   ![image](https://github.com/user-attachments/assets/bc176c08-b7d6-455f-901c-9dac2c2e8e4f)

1. Click the **git-clone** task in the pipeline, and on the properties pane, scroll down to the Workspaces section. From the **output** drop-down list, select **my-workspace** . With this, we have completed the first (git-clone) task.
   
   ![image](https://github.com/user-attachments/assets/eaef6fb8-7f4b-44d3-b974-e1722d7e48f4)

<div id="sec4b"></div>

### 4b. Create s2i-python task

1. Now, let’s create the next task, s2i-python, which helps to build the source code into a Docker image and push it to the quay.io registry. Go back to the pipeline builder view, hover the mouse pointer over the git-clone task, and click the **“+”** sign to the right of the git-clone task to add a new task.

   ![image](https://github.com/user-attachments/assets/d90e73fc-45c1-43cd-a9ad-b6865cadee7c)
   ![image](https://github.com/user-attachments/assets/fd713814-13e0-4b8b-9f37-c4c1b74be709)

1. You can see a new **Select Task** drop-down list created. From this list, select the **s2i-python** task.

   **Note**: In this example, we are selecting the s2i-python task because the application code in the GitHub repository is written in Python language.

   ![image](https://github.com/user-attachments/assets/c3ce5ad2-9f13-48f9-8fcf-f95b015f165c)
   ![image](https://github.com/user-attachments/assets/7d044761-64ce-47ed-a58d-0b5efe467124)

1. Notice that a new **s2i-python** task is created, and is placed after the **git-clone** task.

   ![image](https://github.com/user-attachments/assets/df87dc95-51b7-4f9d-8613-ab877076d4d4)

1. Click the **s2i-python** task, and in the properties pane that is displayed on the right side, and enter the following values for the available fields:

   **IMAGE** = `<`URL of your quay.io repository/`>`:latest (**‘quay.io/dpkshetty/demos:latest’** in my case)

   **Note**: Docker images are always of the form, **`<`name:tag`>`**. The `<`tag`>` field is used to represent the variants of an image (such as different versions, different architecture, different releases, and so on). Here we are using the latest tag to specify that it is the latest version of the Docker image.

   **Workspaces** = <select the workspace from the drop-down list, you earlier created> (‘my-workspace’ in my case).

   Retain the default values for the remaining fields.

   ![image](https://github.com/user-attachments/assets/71a95225-b4c4-40fa-9b69-07dccc314e4c)

1. Click **Create** in the pipeline builder view to create the pipeline with the git-clone and s2i-python tasks.

   ![image](https://github.com/user-attachments/assets/fe3a5de4-8609-45d0-aa21-4e71afc571af)

   The pipeline details page is displayed.

   ![image](https://github.com/user-attachments/assets/0991ec1b-7f49-4015-8224-6d67290297e3)

1. In case you missed to enter data for any of the fields or wish to edit them, click **Actions -> Edit Pipeline**. On the Pipeline builder page, select the task you wish to edit and update its properties. After completing the updates, click **Save** to confirm the changes made.

   ![image](https://github.com/user-attachments/assets/4e500199-2fe0-490d-a57e-7d6d9416945d)
   ![image](https://github.com/user-attachments/assets/0699acf9-4668-405d-945c-ef74ae599060)

<div id="sec5"></div>

### 5. Run the pipeline

1. On the Pipeline details page, click **Actions -> Start**.

   ![image](https://github.com/user-attachments/assets/98bbb870-bfda-4182-b599-1832703de555)

1. On the Start Pipeline page, specify the following values:

   * In the my-workspace field, select **VolumeClaimTemplate**, which automatically creates a PersistentVolumeClaim (PVC) of 1 GiB and provision storage for our workspace area.
   * In the **Advanced options** section, expand **Show Credential options**.

     ![image](https://github.com/user-attachments/assets/9c8925bb-9384-4eb4-832a-33747e9ff8d7)
   * We need to provide the quay.io credentials for the PipelineRun job to be able to access our quay.io account and push the Docker image. The credentials are provided as part of a OpenShift secret. To add the secret, click **Add Secret**.

     ![image](https://github.com/user-attachments/assets/699a78c6-106f-4e78-9457-6d4394ea1903)
   * Enter a name for the secret (**quay-demos** in my case), and in the **Server URL** and **Registry server address** fields, enter **quay.io**.

     Retain the default values for the remaining fields.

     ![image](https://github.com/user-attachments/assets/8b49ffc2-8ff5-4584-955b-f1800b560ba0)
   * Navigate to your quay.io robot account (created in the ‘[Create a new quay.io repository to publish Docker image](#sec3)’ step above) page, click the robot account (**dpkshetty+demos** in my case) and copy the username and password from the quay.io page to the Username and Password fields in the OpenShift console respectively. Click the tick mark symbol to save the secret.

     ![image](https://github.com/user-attachments/assets/d99dfed7-b868-4d97-bbf2-17949e29def3)
     ![image](https://github.com/user-attachments/assets/b2604eac-bf3a-47d3-bc24-216cc46c16ae)
     ![image](https://github.com/user-attachments/assets/2b3c1cb0-fd10-4404-bd6b-83c35a457ba3)

     Notice that the newly secret appears on the Start Pipeline page.

     ![image](https://github.com/user-attachments/assets/2310c392-9224-489e-aeec-77607b4ef697)

1. Ideally at this point, you would click **Start** to run the pipeline. But at the time of writing this tutorial, OpenShift 4.8.x has a small bug in the secret creation process. The secret is malformed and needs to be corrected before we can start running the pipeline. So for now, click **Cancel** (no worries the secret created stays) and return back to the Pipelines page.

   ![image](https://github.com/user-attachments/assets/c7a79813-93a1-448d-a4b0-ddf7b0bcbc0d)

<div id="sec5a"></div>

### 5a. Fix the secret bug (optional)

*(This step is optional and can be skipped if your OpenShift cluster doesn’t have the secret bug)*

1. Click the **Secrets** tab.

   ![image](https://github.com/user-attachments/assets/2c60b5d9-6e81-49d0-95dd-ec587e9b9709) 
1. Search for your secret (**quay-demos** in my case).

   ![image](https://github.com/user-attachments/assets/80b0a695-5a9e-4c0a-ac4b-8551d3d33009) 
1. For your secret entry, click **Edit Secret**.

   ![image](https://github.com/user-attachments/assets/3ed3d66f-16a7-4355-bb59-37cd6c0c093c)
1. Notice that there are multiple redundant and malformed entries in the secret file (that’s the bug). All entries (except the last entry) have the **Username** and **Password** fields empty and the **Registry server address** field incomplete (first entry has q, second entry has qu, and so on).

   ![image](https://github.com/user-attachments/assets/e6b5fed8-d07f-4899-9dff-8d3fe056c540)
   ![image](https://github.com/user-attachments/assets/ba4ce8b5-b84d-497c-9596-7f595a7861f9)
1. In this example, only the last entry (scroll down to the end of the page) is valid, and the rest all are invalid. Click **Remove credentials** for all incorrect entries.

   ![image](https://github.com/user-attachments/assets/ff9082f3-5e1e-439a-bb6b-c875c4bdc184)
1. Notice that there should be only one valid entry with all the fields populated as shown in the following screen capture. Click **Save**.

   ![image](https://github.com/user-attachments/assets/bdbc6401-3908-4f3b-9c79-5613f2ecf276)

<div id="sec5b"></div>

### 5b. Execute the pipeline

1. Click the **Pipelines** tab to view the Pipelines page.

   ![image](https://github.com/user-attachments/assets/ff2d663c-8402-45f9-8cc5-1e4c13ba113c)
1. Click the three vertical dots option next to the pipeline and click **Start**.

   ![image](https://github.com/user-attachments/assets/4fd1a7fc-78fd-4834-aa53-9ee4ce5a6d0d)
1. From the my-workspace drop-down list, select **VolumeClaimTemplate** and click **Show Credential options**. Ensure that the previously created secret (**quay-demos** in my case) exists. Click **Start** to run the pipeline.

   ![image](https://github.com/user-attachments/assets/b0b383bd-a349-4957-9f0b-6cc1a516a022) 
1. On the PipelineRun details page, notice that the first task (**git-clone**) has started.

   ![image](https://github.com/user-attachments/assets/8af024ce-ecef-437c-a6d9-de024420dce3)
1. On the Logs page, view the logs for each step being executed as part of the PipelineRun job.

   ![image](https://github.com/user-attachments/assets/5aefa1a9-04f0-4898-a6e0-d8e99c86a9d6)
   
1. Wait for both the tasks to complete. After successful completion, notice that the status of PipelineRun is **Succeeded**.

   ![image](https://github.com/user-attachments/assets/603ee81b-8107-410a-b50d-f0d6c6fbd5a6) 
   
1. For each task, OpenShift creates a new pod and runs the task steps inside the pod. Click the **TaskRuns** tab to view the task runs associated with this PipelineRun, and the pods associated with each task.

   ![image](https://github.com/user-attachments/assets/4f7de858-43a9-4ee8-a3d0-6c23929c010e)  
   
<div id="sec6"></div>

### 6.	Verify Docker image creation in quay.io

1. Navigate to quay.io and click **Repositories**.

   ![image](https://github.com/user-attachments/assets/3fc8666d-5957-4f22-9bd4-b0c45b979d29)  
1. Click the repository name (**dpkshetty/demos** in my case).

   ![image](https://github.com/user-attachments/assets/799d8a16-825a-4a59-948b-771586c8ec90) 
1. Click the **Tags** icon.

   ![image](https://github.com/user-attachments/assets/be08999c-9cee-41b0-9b59-002ca807dce8)

1. Notice that a new Docker image with tag, **latest** is created few minutes ago. We entered this tag in the s2i-python task’s properties for the Image field (See section, [Create a simple pipeline to build and publish a Docker image from the GitHub source code](#sec4)).

   ![image](https://github.com/user-attachments/assets/a3eaa912-66cc-4d3a-a21b-27d06b943e40)

**Congratulations!** You have successfully created a Docker image in quay.io from the GitHub source code using OpenShift Pipelines.

<div id='sec7'></div>

### 7.	Validate the Docker image created in quay.io

1. To create a new application or pod in OpenShift using this newly created Docker image, navigate to your OpenShift console, click **+Add** and then click **Container Images**.

   ![image](https://github.com/user-attachments/assets/d9f91bcf-e91e-4b59-9519-515a3dd73159)

1. On the Deploy Image page, enter the required values in the following field:

   * **Image name from external registry**: Your quay.io Docker image URL (**quay.io/dpkshetty/demos:latest** in my case). After entering, press the Tab key and wait for OpenShift to validate the URL. You should see the **Validated** message below the URL which ensures OpenShift is able to view and access the Docker image URL.

     ![image](https://github.com/user-attachments/assets/23556087-4b99-4157-9d25-41457825c4ff)

1. In the General section, enter **demos-app** in the **Application Name** field, and **demos** in the **Name** field.
1. In the Resources section, select **Deployment** as the resource type to generate, and in the Advanced options section, select the **Create a route to Application** checkbox.

     ![image](https://github.com/user-attachments/assets/0b620e44-53ae-45ee-ad2b-b7a0f623e3b4) 
1. Optionally, specify the options for a secure route (refer to the following note for details).

   **Note**: The steps to add a secure route can be skipped if you are using an OpenShift cluster where HTTP routes are allowed. In my case, OpenShift on IBM Power Virtual Server mandates to use HTTPS (secure HTTP) routes and plain HTTP routes **are not supported**. Hence, I need to perform the following steps. If unsure, Check with your cluster administrator for further details.

   * Expand **Show advanced Routing options**.

     ![image](https://github.com/user-attachments/assets/317d068d-d52b-4181-bf2a-16b1b114458c)

   * Select the **Secure Route** checkbox.

     ![image](https://github.com/user-attachments/assets/b9a2cec1-38be-487d-b1c8-161582303ce2)  
   * From the **TLS Termination** drop-down list, select **Edge**, and from the **Insecure traffic** drop-down list, select **None**.

      ![image](https://github.com/user-attachments/assets/7512f65b-d45d-4807-8372-896017f46365)

1. Click **Create**.

   ![image](https://github.com/user-attachments/assets/205d02fe-9e7a-47ca-893f-5dd7ea2f9810)

1. On the Topology view, you can see an icon for your application being deployed. Click the deployment (**D demos**) and in the corresponding properties pane, click the **Resources** tab and wait for pod to be in **Running** state.

   ![image](https://github.com/user-attachments/assets/375b8374-e3e8-4dd2-ae0c-94099ccbc855)

1. In the Routes section, click the location URL.

   **Note**: *Depending on how your OpenShift cluster is configured, you may have a HTTPS or HTTP route (as explained earlier)*.

   ![image](https://github.com/user-attachments/assets/ef64ef78-1f3d-4133-963a-3559cb7d7014)

1. After successful completion, notice the welcome message from the Pyflask app in your browser window.

   ![image](https://github.com/user-attachments/assets/2ee6355a-83b4-4c89-9053-ed7d99cd761f)

1. Also, check the other endpoints (such as **/test** and **/version** by appending it to the end of the URL) to validate if the entire application is working as expected.

   ![image](https://github.com/user-attachments/assets/063d66fd-dc9e-45da-b983-87fa5ab96593)
   ![image](https://github.com/user-attachments/assets/ef650923-4b7b-469a-8710-b80647ac8d47)

**Congratulations!** The Docker image you have created using OpenShift Pipelines is working successfully. You can now share your quay.io Docker image URL (**‘quay.io/dpkshetty/demos:latest’** in my case) with anyone in the world to create or run applications from your Docker image.

<div id='sec8'></div>

### 8.	Automate Docker image build using OpenShift Pipeline triggers and GitHub webhooks

Triggers capture the external events and process them to extract key pieces of information.

A PipelineRun job must run automatically for any new code changes in the Git repository and that is how we can achieve automation of the pipeline we created earlier using OpenShift Pipelines.

Triggers automate this process by capturing and processing any change event and by triggering a PipelineRun job that deploys the new image with the latest changes from your Git repository.

<div id='sec8a'></div>

### 8a. Set up pipeline triggers

1. In the Pipelines view click the three vertical dots icon next to the pipeline, and click **Add Trigger**.

   ![image](https://github.com/user-attachments/assets/4f94ad64-81dd-4ac3-a091-e2fb288644bd)
   ![image](https://github.com/user-attachments/assets/5a82d142-a6c1-476b-8b64-5c4e026f5116) 
1. On the Add Trigger page, enter the values for the following fields:
   * From the **Git Provider type** drop-down list, select **github-push**.
   * From the **my-workspace** drop-down list, select **VolumeClaimTemplate**.

   Then click **Add**.

   ![image](https://github.com/user-attachments/assets/2966ebff-cba7-4299-8dec-43b0e92a51f1)  
1. On the Pipelines page, click the pipeline.

   ![image](https://github.com/user-attachments/assets/bf7c36c2-89ad-47ee-b6ec-32d386e470f6)  
1. On the Pipeline details page, you can see that the event listener HTTP route URL has been created. Event listener is a component of pipelines trigger that listens to the external events.

   ![image](https://github.com/user-attachments/assets/1fd93651-ef48-4e39-b8c6-403177432186)  
1. Copy the HTTP URL (applicable only if your OpenShift cluster supports HTTP routes, and if not, navigate to the **Create a event listener HTTPS route** section and copy the HTTPS URL) and save it for later use. This URL (HTTP or HTTPS as applicable) will be used as a payload URL in GitHub webhooks setup (you can find details in the subsequent topics).

<div id='sec8b'></div>

### 8b. Create a event listener HTTPS route URL (optional)

**Note**: The steps to add a secure event listener route can be skipped if you are using an OpenShift cluster where HTTP routes are allowed. In my case, OpenShift on Power Virtual Server mandates to use HTTPS (secure HTTP) routes and plain HTTP routes **are not supported**. Hence, I need to perform the following steps:

1. Switch to **Administrator** persona, and click **Networking -> Routes**. On the Routes page, you can see a route entry, named **el-event-listener-xxx**, representing the event listener object, the associated HTTP route URL, and the corresponding event listener service object to which this route maps to.

   ![image](https://github.com/user-attachments/assets/3be8cde8-4827-4d7b-bcee-9ac82f30bdea)  
1. Click **Create Route** and enter the required values for the following fields:

   * In the **Name** field, enter **my-https-route** (or any name as you wish).
   * From the **Service** drop-down list, select the existing **el-event-listener-xxx** service.
   * From the **Target port** drop-down list, select **8080 -> 8080 (TCP)**.
   * Select the **Secure Route** checkbox. In the same section, from the **TLS termination** drop-down list, select  **Edge** and from the **Insecure traffic** drop-down list, select **None**.

     Retain the default values for the remaining fields, scroll down, and click **Create**.

     ![image](https://github.com/user-attachments/assets/33eaf38c-8c65-4a76-81f6-50fdf27396bc)
     ![image](https://github.com/user-attachments/assets/719974a3-7b52-42c4-8dfa-aadf45b13682)
     ![image](https://github.com/user-attachments/assets/83e56df7-8efc-4bdf-b3fe-00ec0876737c)

1. Notice that a new route has been created with the HTTPS route URL.

     ![image](https://github.com/user-attachments/assets/65e053a4-b52f-4964-b1c9-61b42e12eaed)  
1. Save this HTTPS URL for later use.

<div id='sec8c'></div>

### 8c. Set up GitHub webhooks

GitHub webhooks allow external services to be notified when certain GitHub events happen. We are interested in the **git push** event. In this procedure, we will configure GitHub webhooks with the event listener HTTP or HTTPS (as applicable) URL as the payload URL, such that changes to the GitHub source code will be notified to event listener which will help trigger a newPipelineRun job.

1. Navigate to your GitHub repo in browser (<a href="https://github.com/dpkshetty/pipeline-s2i-pyflask-demo" target="_blank" rel="noopener noreferrer">https://github.com/dpkshetty/pipeline-s2i-pyflask-demo</a> in my case, and your URL will be different).

   ![image](https://github.com/user-attachments/assets/8cc1858b-0978-4416-b86f-c75a9d63896b)  
1. Click the **Settings** tab, and then click **Webhooks**.

   ![image](https://github.com/user-attachments/assets/512d5993-3e50-4913-8248-828c8f332bc1) 
1. Click **Add webhook**.

   ![image](https://github.com/user-attachments/assets/1b809e40-e735-42b6-b822-6595dfb89266)  
1. On the Add webhook page, enter the necessary data in the following fields:

   **Note**: GitHub might prompt you to authenticate one more time. if so, log in with your GitHub credentials.

   In the **Payload URL** field, enter the HTTP or HTTPS (as applicable) event listener route URL (in my case it is a HTTPS URL).

   From the **Content type** drop-down list, select **application/json**.

    You can retain the default values for the remaining fields, and then click **Add webhook**.

    ![image](https://github.com/user-attachments/assets/a64ddf86-cc83-49c8-b1f3-e58b6b339c09)
    ![image](https://github.com/user-attachments/assets/1ddc1e24-b80d-4131-99af-912f205ec069)

1. A new webhook will be created (the last entry in case you have multiple webhooks). It will  have a tick-mark beside it (you may have to refresh the webhooks page in case you don’t see it automatically), which indicates that GitHub is able to ping or connect to your OpenShift cluster using the event listener HTTP or HTTPS (as applicable for your cluster) route URL.

    ![image](https://github.com/user-attachments/assets/a365e97e-225e-4fe5-9a08-53d0fe760829)

**Congratulations!** GitHub webhook is now connected with your OpenShift cluster and any changes to the GitHub source code will trigger a newPipelineRun job.

<div id='sec9'></div>

### 9.	Verify if a GitHub code change creates a new Docker image

Let’s make a small code change in our GitHub repo and check if it indeed creates a new PipelineRun job, which in turn creates a new Docker image of our application.

1. Navigate to your GitHub source repo, click **app.py**, and edit it by clicking the pencil icon.

    ![image](https://github.com/user-attachments/assets/63a8c3e0-4ec3-443e-ab48-5604b70193f9)  
1. In the edit mode, make the following two changes:

   * Modify  the welcome message by adding **Pipelines** to make it ‘Pyflask Pipelines Demo’.
   * Upgrade the version to 2.0.
    ![image](https://github.com/user-attachments/assets/3d343e1a-bff7-4bf6-8cf5-275edbb390d6)

1. Scroll down to add a brief description for the changes made and click **Commit Changes**.

    ![image](https://github.com/user-attachments/assets/3b77d26c-14b1-4afe-a4fa-c28081fc09e0)
1. When committed successfully, a new PipelinRun job is triggered. Navigate to your OpenShift console, click **Pipeline**, and then click your pipeline. In the Pipeline details view, click the **PipelineRuns** tab.

    ![image](https://github.com/user-attachments/assets/592abc3b-713f-421f-9b6d-978c9d2b24f0)  
1. Notice that a new PipelineRun job is running.

    ![image](https://github.com/user-attachments/assets/c772b007-3f6c-4f51-a91d-05785261eb91) 
1. Wait for the PipelineRun job to finish.

   **Note**: While it is running, you may want to click the new PipelineRun job and view the logs to monitor the git-clone and s2i-python tasks that run as part of this new job (as we did before).

    ![image](https://github.com/user-attachments/assets/0682f323-6b19-4ec8-8e97-4df9394f7dae)  
1. Switch to quay.io in your browser and verify that a new Docker image is pushed to the registry.

   **Note**: If you already have the quay.io registry page opened, refresh the page to see the latest information.

   Notice that a new image is just pushed!

    ![image](https://github.com/user-attachments/assets/0a20db64-ef7d-4970-a164-0e693828741f)

**Congratulations!** You have successfully used GitHub webhook and the pipelines trigger functionality to auto build and deploy a new Docker image in the event of a GitHub source code change.

<div id='sec10'></div>

### 10.	Validate if the new version of the Docker image works

Perform the steps mentioned in the [Validate the Docker image created in quay.io](#sec7) section to create a new application and verify that the application has the new code changes. Refer to a sample in the following screen capture.

The new welcome message:

![image](https://github.com/user-attachments/assets/de729944-db82-42d7-8c3b-29cdf1773f8a)

The new version of the application:

![image](https://github.com/user-attachments/assets/4feb7c4e-0bad-4493-a2b5-a70f82a33fa6)

**Congratulations!** This verifies that the new Docker image built from an automated PipelineRun job triggered by the GitHub source code change reflects the updates!

## Summary

In this tutorial, you learnt how to create a simple pipeline to build and deploy a Docker image from the GitHub source code by mainly using the OpenShift GUI and without YAMLs and CLIs! You also learnt how to use the Pipeline Trigger functionality along with GitHub webhooks to automate the Docker image creation process. DevOps engineers are always looking to automate their software build, test, deploy lifecycle and pipelines provide an excellent way to automate the software development lifecycle.

While we hardcoded the GitHub repo and the quay.io repository URLs for this tutorial (to keep things simple) it is possible to enhance the pipeline further by generalizing it. You can use pipeline parameters (also known as params) as an input method for specifying the GitHub and quay.io repository URLs and thus make the pipeline more reusable by providing the ability to specify GitHub and quay.io repository (instead of hardcoding it) as part of each pipeline run.

I will leave that as a recommended exercise for anyone interested. As a hint, you need to use the **Add Parameter** option in the “Parameters” section of pipeline builder page to create new parameters and reference them using the *$(params.`<`param-name`>`)* syntax when populating the Tasks’ properties. Refer to the <a href="https://docs.openshift.com/container-platform/4.8/cicd/pipelines/working-with-pipelines-using-the-developer-perspective.html" target="_blank" rel="noopener noreferrer">Red Hat OpenShift documentation</a> for more details. Good luck!

## Acknowledgment

I would like to thank <a href="https://www.linkedin.com/in/sebastien-chabrolles-50448520/" target="_blank" rel="noopener noreferrer">Sebastien Chabrolles</a> for helping with queries specific to pipelines and issues I encountered while creating this tutorial, and especially helping in mitigating the secret bug which was causing the PipelineRun job to fail.

## Take the next step

Join the <a href="https://community.ibm.com/community/user/powerdeveloper/home" target="_blank" rel="noopener noreferrer">Power Developer eXchange Community (PDeX)</a>. PDeX is a place for anyone interested in developing open source apps on IBM Power. Whether you're new to Power or a seasoned expert, we invite you to join and begin exchanging ideas, sharing experiences, and collaborating with other members today!

---
draft: false
ignore_prod: false
display_in_listing: true
title: Build multi-architecture container images using GitLab
subtitle: "Code once, build for multiple architectures automatically using GitLab CI"
meta_title: Build multi-architecture container images using GitLab
authors:
  - email: deepakcshetty@in.ibm.com
    name: Deepak C Shetty
  - email: s.chabrolles@fr.ibm.com
    name: Sebastien Chabrolles
completed_date: '2023-03-07'
check_date: "2025-02-01"
last_updated: '2023-04-20'
excerpt: "Code once, build for multiple architectures automatically using GitLab CI"
meta_description: "Code once, build for multiple architectures automatically using GitLab CI"
meta_keywords: 'GitLab, CI, multi-arch, DevOps, Red Hat OpenShift, IBM Power, IT Infrastructure, Linux on IBM Power'
primary_tag: ibm-power
tags:
  - infrastructure
  - ci-cd
  - devops
components:
  - linux-on-ibm-power
  - redhat-openshift-ibm-cloud
also_found_in:
  - "learningpaths/exploring-openshift-powervs/"
---
## Introduction

GitLab is a web-based Git repository that provides free public and private Git repositories, issue-following capabilities, and wikis. It is a complete DevOps platform that enables developers to perform all the tasks in a project—from project planning and source code management to monitoring and security. It allows teams to collaborate and build better software.

One of the most important features GitLab provides is continuous integration/continuous deployment (CI/CD), which developers can use to build, test & deploy their software whenever they push any code changes to their application.

In this tutorial, we will use GitLab to create a small CI pipeline which will create container images of our sample application for ppc64le (IBM Power) and x86 (Intel) hardware architectures concurrently and push the images to the Quay.io container registry. Because container images are hardware (HW) specific, we need one image per HW architecture. This causes issues while automating and/or sharing images because we need to know the HW architecture beforehand in order to pick/serve the right image for our application to be deployed successfully across different Red Hat OpenShift clusters.

This tutorial shows you how to solve the multi-architecture multi-image problem by creating container manifests that will automatically serve the right container image based on the OpenShift cluster’s HW architecture. This ensures we need to deal with only one container image across OpenShift clusters of different HW architectures.

**Solution architecture**

The solution architecture for creating a multi-arch container image using GitLab CI pipeline is depicted below:

##### Figure 1: Using Gitlab-CI pipeline across multiple OpenShift/Kubernetes clusters.

![Figure 1](https://github.com/user-attachments/assets/95db4018-38f2-4c80-b93f-32aee9c29d33)

As depicted in the picture above, the GitLab Runner applications installed in the respective OpenShift clusters connect with the GitLab server hosting your application code and GitLab CI pipeline (**.gitlab-ci.yml**). Learn more about <a href="https://docs.gitlab.com/runner/" target="_blank" rel="noopener noreferrer">GitLab Runner</a>.

The CI pipeline gets triggered whenever a change is made to the pipeline itself and/or application code. This pipeline trigger will cause the OpenShift cluster to build the HW architecture-specific container image(s)—x86 image and ppc64le image in our case—and push them to the container registry (Quay.io in this case). Eventually, the pipeline will combine the different (HW-specific) container images and create a multi-architecture (single) image which can be used across x86 and ppc64le OpenShift clusters. This saves the developers and operations team from dealing with multiple container images for an application.

This tutorial defines each of the above steps while providing detailed explanation of the CI pipeline YAML file and how it works. Designed to be simple to follow, this tutorial uses only the OpenShift GUI/console as much as possible.

## Prerequisites

The following are the prerequisites for this tutorial:

* Because we are building multi-arch container images, we need access to two Red Hat OpenShift Container Platform (OCP) clusters, running on different HW architectures. In this example, I have used OpenShift version 4.10.xx on IBM Power Virtual Server (PowerVS) – **ppc64le arch** and OpenShift version 4.10.xx on VMware on IBM Cloud – **x86_64 arch**.
* Both the clusters will need cluster-admin access, as we intend to install the GitLab Runner Operator, which is needed to run GitLab CI pipelines, unless it is pre-installed.
* Valid login credential is required to access GitLab and Quay.io.
* Familiarity with Quay.io and GitLab is a must.

## Estimated time

One hour to create a new GitLab pipeline, configure it, and execute it to create a single multi-architecture Docker image.

## Steps

1. [Setup GitLab repo and Quay.io credentials](#step1)
1. [Install GitLab runner operator in OpenShift](#step2)
1. [Create GitLab Runner instance in OpenShift](#step3)
1. [Verify Runner connection with GitLab server](#step4)
1. [Provide right permissions to the OpenShift service account](#step5)
1. [Configure GitLab server with quay.io credentials](#step6)
1. [A peek at the GitLab CI pipeline yaml file](#step7)
1. [Customize CI pipeline to suit your environment](#step8)
1. [Understanding the GitLab CI Pipeline yaml file](#step9)
1. [Executing the GitLab CI Pipeline](#step10)
1. [Verify Docker image creation in Quay.io registry](#step11)
1. [Validate the multi-arch Docker image created in Quay.io](#step12)
1. [Troubleshooting](#step13)

<div id="step1"></div>

### Step 1. Setup GitLab repo and Quay.io credentials.

1. I’ll be using <a href="https://gitlab.com/dpkshetty/gitlab-s2i-pyflask-demo" target="_blank" rel="noopener noreferrer">my GitLab repo</a>

   Because this is a public repo, readers interested in following this tutorial should *clone/import* this repo into their respective GitLab profile as we need author permissions to edit/update files in the repo while following the steps in this tutorial.

1. I’ll be using Quay.io for the container repository, where we’ll store our container images.For more details, refer to <a href="https://quay.io/repository/dpkshetty/demos" target="_blank" rel="noopener noreferrer">my Quay.io repository</a>. Readers interested in following this tutorial should *create their own Quay.io repository* to use with this tutorial. They should also *create a robot account for their repository*. The robot account should have write permissions to their repository (“demos” in my case). Save the robot account credentials in a safe place as you’ll need it while specifying the credentials to push the image to this repository via the GitLab CI pipeline (covered later).

   Note: If you are new to using Quay.io – you can refer to the <a href="https://developer.ibm.com/tutorials/build-docker-images-from-github-source-using-openshift-pipelines/#sec3" target="_blank" rel="noopener noreferrer">step 3 in this tutorial</a> on how to create a new Quay.io repository and create robot account to provide write permissions.

<div id="step2"></div>

### Step 2. Install GitLab Runner Operator in OpenShift.

Note: The steps below are shown for the IBM Power (ppc64le arch) OpenShift cluster. Repeat the same on the Intel (x86) cluster.

1. Switch to the **Administrator** persona.
1. Click **OperatorHub** and  in the text field, enter **runner**.
1. Click the first **GitLab Runner** listing (ignore the community version of it).

   ![image](https://github.com/user-attachments/assets/27b2768f-2ed5-4c1e-9b22-5f4220de5958)

1. Click **Install**.

   ![image](https://github.com/user-attachments/assets/d0c864ab-3844-4a18-8aa2-a18d9c989343)

1. On the **Install Operator** screen, ensure that **All namespace on the cluster (default)** is selected. Retain everything else to default values and then click **Install**.

   ![image](https://github.com/user-attachments/assets/63bc2f91-169d-4169-b54a-b662c95f20ee)


1. Wait for the GitLab Runner Operator to get installed. It takes a few minutes. Click **Installed Operators** and on the **Installed Operators** page notice that **GitLab Runner** is now displayed, indicating successful installation.

   ![image](https://github.com/user-attachments/assets/f1ef55a7-797e-4e7a-bb61-3807dde953ad)

Congratulations! You have successfully installed GitLab Runner Operator

<div id="step3"></div>

### Step 3. Create GitLab Runner instance in OpenShift.

Note: The steps below are shown for IBM Power (ppc64le arch) OpenShift cluster. Repeat the same on the Intel (x86) cluster.

1. Create a new project. Click **Projects** and then click **Create Project**.

   ![image](https://github.com/user-attachments/assets/43f7bf8f-9c0f-4fec-b748-2fea226dc3cb)

1. I’m creating a new project named **tutorial**. Enter **tutorial** in the **Name** field and click **Create**.

   ![image](https://github.com/user-attachments/assets/480518b2-9516-418c-b613-8fe08a85a783)

1. Switch to the **Installed Operators** view. Ensure that you are in the **tutorial** project, if not, click the **Project** drop-down list and select **tutorial**.

   ![image](https://github.com/user-attachments/assets/cfd0f675-2aa2-4518-9dfe-f519c27b53f9)

1. GitLab Runner needs a registration token to connect and authenticate with the GitLab server. So, lets create a secret with key/value pair to hold the GitLab registration token. Go to your GitLab project, click **Settings**, and then click **CI/CD**.

   ![image](https://github.com/user-attachments/assets/1f23d93c-f5c2-4e07-af96-cd8d9760601f)

1. In the **Runners** section, click **Expand**.

   ![image](https://github.com/user-attachments/assets/3883d7a0-1a52-4873-9561-8d7347f34dd4)

1. Scroll to the **Specific runners** section and copy/save the token for future use. This is the token that will be used by the runner instance (which we will create further in the tutorial) to register/connect with the GitLab server. We don’t plan to use **Shared runners** in this tutorial, so disable them.

   ![image](https://github.com/user-attachments/assets/55f3c7ce-0927-4b24-b088-3bcfb3e08457)

   ![image](https://github.com/user-attachments/assets/be2728c3-7e6e-407e-8163-2bbe7a58fba6)

1. Now let’s create a secret to hold this registration token. In the OpenShift console, click **Workloads -> Secrets**. From the **Create** drop-down list,select **Key/value secret**. Ensure that you are in the **tutorial** project namespace, as the secret is created for this project only!

   ![image](https://github.com/user-attachments/assets/c1020a5d-b971-4710-afdc-5316fb778c18)

1. In the form that appears, fill the fields as below:

   **Secret name**: **dpk-gitlab-secret**  
   **Key**: **runner-registration-token**  
   **Value**: *<copy the registration token saved in step 3.6 here>*

1. Click **Create**.  

   ![image](https://github.com/user-attachments/assets/965a8c4e-9a3b-4071-b08c-ab7d360b1356)

   Congratulations! You have created the secret successfully!

1. Now, let’s create a new service account in OpenShift for use with GitLab Runner. We want to create our own service account to make it easy to assign the right roles and privileges for the runner to get enough permissions to run our CI pipeline tasks. Go back to your OpenShift console and click **User Management** -> **ServiceAccounts** and then click **Create ServiceAccount**.

   ![image](https://github.com/user-attachments/assets/c50c76d0-f89d-4698-95ca-71ddf9efec7b)

1. Unfortunately, there is no GUI way of doing this. In the YAML page displayed, replace the default name **example** with **dpk-gitlab-sa** and click **Create**. This is the name of our service account (the following screen captures show the YAML file before and after the default name change).

   Before name change:

   ![image](https://github.com/user-attachments/assets/4ad42e56-1e80-4dba-be48-182425740326)

   After name change:

   ![image](https://github.com/user-attachments/assets/a2dba0bc-080d-415d-a471-44b52c22eeec)

    Congratulations! You have successfully created a service account.

1. Now we are ready to create a GitLab Runner instance. Click **Operators** -> **Installed Operators** and click  **GitLab Runner**.

   ![image](https://github.com/user-attachments/assets/b658021b-6d2a-4fdf-bd87-19e5f50c5010)

1. Click the **GitLab Runner** tab and then click **Create Runner**

   ![image](https://github.com/user-attachments/assets/04865fab-173b-4f13-bb97-1b50a0947f0c)

1. In the resulting form, fill the fields as below:

   **Name**: **dpk-ppc64le**

   (Note: I am using a ppc64le cluster, and hence the name. For the x86 cluster, I will have dpk-x86, so that it is easy to differentiate the runners from one another in the GitLab logs.)

   **GitLab URL**: **`https://gitlab.com`** (leave as default)

   **Registration Token**: **dpk-gitlab-secret** (the secret we created in step 3.8)

   **Concurrent**: **10**

   **Tags**: **openshift**, **ppc64le**  
   (Note: For the x86 cluster, the tags would be **openshift**, **x86**)
   
   **Serviceaccount**: **dpk-gitlab-sa** (the service account we created in step 3.10)  
   Retain the default values for all other fields.

1. Click **Create**.

   ![image](https://github.com/user-attachments/assets/ec9c5439-c5cb-4b45-a2af-bec3e88c2b39)

   ![image](https://github.com/user-attachments/assets/3a78cd9c-b5e4-4d85-ae1e-44306639d676)

   ![image](https://github.com/user-attachments/assets/02769e34-5045-48cd-90c6-600e55a6b546)

1. In the **Runners** view, notice that the status shows  **Pending** for some time, and later changes to **Running**. Note that it may take few minutes to change from **Pending** to **Running**, implying that the runner instance was successfully able to connect with the GitLab server (which we will verify in the next step).

   ![image](https://github.com/user-attachments/assets/1debeecc-72be-45d0-8d41-bae16400bf16)

Congratulations! You have successfully installed GitLab Runner Operator and created a GitLab runner instance.

<div id="step4"></div>

### Step 4. Verify runner connection with the GitLab server.

Note: The steps below are shown for the IBM Power (ppc64le) OpenShift cluster. Repeat the same for the Intel (x86) cluster.

1. Navigate to your GitLab repository (mine is <a href="https://gitlab.com/dpkshetty/gitlab-s2i-pyflask-demo" target="_blank" rel="noopener noreferrer">https://gitlab.com/dpkshetty/gitlab-s2i-pyflask-demo</a> ), click **Settings** -> **CI/CD**.

   ![image](https://github.com/user-attachments/assets/7eb032ca-69fa-4cf6-a2e1-b07e77a1692a)

1. Scroll down to the **Runners** section and click **Expand**.

   ![image](https://github.com/user-attachments/assets/ff08da51-7fb3-4c98-9a2d-406588d5b4d0)

1. You should be able to see your runner’s information (**dpk-ppc64le-xxx** in my case) in the **Specific runners** section, along with the tags you specified while creating the runner instance (**openshift**, **ppc64le**). This confirms that your OpenShift runner instance is connected with the GitLab repository successfully.

   ![image](https://github.com/user-attachments/assets/2d3b02ff-a4b8-4c21-b2e1-b2530f6ee747)

Note: Repeat the above steps for the x86 cluster as well. You should be able to see an additional runner instance with **openShift, x86** as the tags (refer to the following screen capture)

Congratulations! You have successfully verified the connection between your OpenShift Runner instance and GitLab repository.

<div id="step5"></div>

### Step 5. Provide the right permissions to the OpenShift service account.

Let’s provide adequate permissions for this service account to be able to run CI tasks (which are containers themselves). We will associate this service account with the **gitlab-runner-app-role** and with the anyuid **cluster-role**. This ensures the service account has enough privileges to create different OpenShift objects/resources needed to run CI task containers with the right privileges.

1. Click  **User Management** -> **RoleBindings** and then click **Create binding**.

   ![image](https://github.com/user-attachments/assets/a4f7a373-fecc-47ad-a38c-4f8ba4f5e113)

1. In the resulting form, fill in the following fields with the given values:

   **Binding type**: **Namespace role binding**

   **Name**: **add-anyuid-to-my-gitlab-sa** (you can enter any name of your choice)

   **Namespace**: **tutorial**

   **Role name**: **system:openshift:scc:anyuid**

   **Subject**: **ServiceAccount**

   **Subject namespace**: **tutorial**

   **Subject name**: **dpk-gitlab-sa** (the service account we created in step 3.10 )

1. Click **Create**.

   ![image](https://github.com/user-attachments/assets/1753dea6-2bf7-4103-b204-99a8c95a6faa)

   The **anyuid** role binding provides the service account with the privilege to run container as any UID including root.

1. Click **RoleBindings**. Then click **Create binding**.

   ![image](https://github.com/user-attachments/assets/4ef21b51-95e1-41ea-b95f-c02a998e3b05)

1. Create one more role binding to provide our service account with adequate privileges to create OpenShift objects/resources which are needed while running the CI pipeline. In the resulting form, fill in the following fields with the given values.

   **Binding type**: **Namespace role binding**

   **Name**: **add-my-gitlab-sa-to-runner-app-role** (you can enter any name of your choice)

   **Namespace**: **tutorial**

   **Role name**: **gitlab-runner-app-role**

   **Subject**: **ServiceAccount**

   **Subject namespace**: **tutorial**

   **Subject name**: **dpk-gitlab-sa** (the service account we created in step 3.10)

1. Click **Create**.

   ![image](https://github.com/user-attachments/assets/2607a272-a5db-4cb6-b4d0-514d47f69de9)

Congratulations! You have successfully provided adequate privileges to the service account user to be able to run CI tasks (which are containers themselves) and execute the pipeline without any constraints.

<div id="step6"></div>

### Step 6.	Configure GitLab server with Quay.io credentials.

In this tutorial, while using Quay.io as the Docker image repository, we’ve already created our own Quay.io repository and created a robot account with write permissions to the repository (refer Step 1.2)

The GitLab CI pipeline needs to know the user and password credentials to be able to push the Docker images that get created as part of the pipeline job execution. Let’s create a protected variable in our GitLab repository, to store the Quay.io repository’s username and password credentials. These credentials will be referenced in the actual pipeline code which we will review shortly.

1. In your GitLab repository, click **Settings** -> **CI/CD**.
1. In the **Variables** section, click **Expand**.

   ![image](https://github.com/user-attachments/assets/5686fb80-66ff-4dbd-a3cf-2396e184dae1)

1. Click **Add variable**.

   ![image](https://github.com/user-attachments/assets/1c7154c8-eb78-4f77-9e52-0b97106c4d3d)

1. In the resulting form, fill in the following fields with the given values.:

   **Key**: **quay_user** (Use the same name as the CI pipeline code uses this variable)

   **Value**: *<Paste your Quay.io repository's robot account's (created in Step 1.2) username>*

   **Flags**: **Protect variable** (selected by default) and **Mask variable**

1. Click **Add variable**.

   ![image](https://github.com/user-attachments/assets/5db8b357-1b38-494a-bdc6-3f2b3fd89b53)

1. To store the password, repeat the previous steps. Click **Add variable** again, and in the resulting form, fill in the following fields with the given values.

   **Key**: **quay_passwd** (Use the same name as the CI pipeline code uses this variable)

   **Value**: *<Paste your Quay.io repository's robot account's (created in Step 1.2) password>*

   **Flags**: **Protect variable** (selected by default) and **Mask variable** (select this option)

1. Click **Add variable**.

   ![image](https://github.com/user-attachments/assets/b2c458f3-dd24-4464-9195-d22d6b2d2174)

1. You should end up with two variables as seen below. These two variables will be referenced in the CI pipeline YAML which we will review shortly.

   ![image](https://github.com/user-attachments/assets/a8d76053-18c7-4016-8165-fd65b3b4538d)

Congratulations! You have successfully configured GitLab to be able to connect to your Quay.io Docker image repository.

<div id="step7"></div>

### Step 7. A peek at the GitLab CI pipeline YAML file.

1. In your GitLab repository (the one you cloned in step 1.1), click on the **.gitlab-ci.yml** file. This is GitLab’s pipeline YAML file and it is pre-populated with a basic pipeline that will take the sample **pyflask** application source code from the GitLab repository, build x86 and ppc64le architecture Docker images and then combine both the images to create a multi-arch manifest image, all of which are stored in the Quay.io image repository.

   ![image](https://github.com/user-attachments/assets/e7d92ddd-d570-49c2-a8ba-eb9d91f93a67)

1. Notice the CI pipeline code hosted in the **.gitlab-ci.yml** file. There are broadly 5 sections, and I’ll explain what each section does (covered later).

   ```
   stages:
      - container-build
      - multiarch-push
   variables:
   IMAGE_REGISTRY: "quay.io/dpkshetty/demos"
   TAG: "gitlab-pyflask"
   APP: dpk-pyflask
   x86-build:
   stage: container-build
   tags:
      - OpenShift
      - x86
   image:
      name: quay.io/containers/podman
   script:
      - echo "Building container on x86"
      - podman --storage-driver=vfs build --isolation chroot -t $APP -f ./Dockerfile --no-cache
      - echo "Build complete."

      - echo "Listing podman images"
      - podman --storage-driver=vfs images

      - echo "Push image to ${IMAGE_REGISTRY}:${TAG}"
      - podman --storage-driver=vfs push --creds $quay_user:$quay_passwd $APP ${IMAGE_REGISTRY}:${TAG}-x86
   ppc64le-build:
   stage: container-build
   tags:
      - OpenShift
      - ppc64le
   image:
      name: quay.io/containers/podman
   script:
      - echo "Building container on ppc64le"
      - podman --storage-driver=vfs build --isolation chroot -t $APP -f ./Dockerfile --no-cache
      - echo "Build complete."

      - echo "Listing podman images"
      - podman --storage-driver=vfs images

      - echo "Push image to ${IMAGE_REGISTRY}:${TAG}"
      - podman --storage-driver=vfs push --creds $quay_user:$quay_passwd $APP ${IMAGE_REGISTRY}:${TAG}-ppc64le
   multiarch-manifest:
      stage: multiarch-push
      tags:
         - OpenShift
      image:
         name: quay.io/containers/podman
      needs:
         - x86-build
         - ppc64le-build
      script:
         - echo "Creating multiarch manifest ${IMAGE_REGISTRY}:${TAG}-multiarch"
         - podman manifest create mylist
         - podman manifest add mylist ${IMAGE_REGISTRY}:${TAG}-ppc64le
         - podman manifest add mylist ${IMAGE_REGISTRY}:${TAG}-x86
         - podman manifest push --creds $quay_user:$quay_passwd mylist ${IMAGE_REGISTRY}:${TAG}-multiarch
   ```

<div id="step8"></div>

### Step 8. Customize CI pipeline to suit your environment.

Before we dig deep into understanding the pipeline YAML file, let’s customize the pipeline YAML file to your environment. Thankfully the only section that needs to be edited/updated is the **variables** section. It is currently populated with my environment details as shown.

1. Please edit/update the fields of the **.gitlab-ci.yml** file in your repository to customize it to for your environment:

   * **IMAGE_REGISTRY**: **quay.io/dpkshetty/demos**
     * Edit this and make it point to `<`*your*`>` Quay.io repository, the one which you created in Step 1.2. I am using **demos** as my repository.

   * **TAG**: **gitlab-pyflask**
     * This is used as the tag for the Docker image being created as part of the container-build stage of the pipeline. You can give any tag of your choice . As you will learn further in the tutorial, the repository name (**demos** in my case) and the tag (**gitlab-pyflask** in my case) is used to create the architecture-specific Docker image (**demos:gitlab-pyflask-x86** and **demos:gitlab-pyflask-ppc64le** in my case) stored in the Quay.io repository.
   * **APP: dpk-pyflask**
     * This is the name of the OpenShift deployment resource/object. You can pick any name of your choice . This gets used as the prefix for all the OpenShift deployments and its associated resources that gets created during the pipeline execution job.

   

Congratulations! You have successfully customized the GitLab CI pipeline YAML file to suit to your environment.

<div id="step9"></div>

### Step 9.	Understanding the GitLab CI Pipeline YAML file.

The **.gitlab-ci.yml** file has 5 sections. To get an understanding of how the pipeline works, let’s look at what each section does.

1. Section 1: **stages**

   This section defines the pipeline stages. We have two stages:

   * **container-build**, which builds the Docker image and pushes it to the Quay.io repository.
   * **multiarch-push**, which creates the multi-arch image and pushes it to the Quay.io repository.

     ![Figure 37](images/fig37.png)

1. Section 2: **variables**

   This section defines the variables we use in other sections of the YAML file.

   * **IMAGE_REGISTRY** – URL of your Quay.io repository to store the Docker images created as part of the pipeline job.
   * **TAG** – image tag to use for the Docker images.
   * **APP** – Prefix used to name all the OpenShift objects/resources created as part of pipeline job.

     ![Figure 38](images/fig38.png)

1. Section 3: **ppc64le-build**

   This section builds the container image (Docker image) for the ppc64le (IBM Power) architecture. This section has four components, explained below:

   * **stage** – specifies the pipeline stage which this section is part of.
   * **tags** – these are the tags which are used to select which OpenShift cluster will be picked to execute the pipeline job. These tags are used to pick the matching OpenShift Runner instance, hence we need to tag the runner instance appropriately when creating it in OpenShift (which we did in Step3.14)
   * **image** – specifies the Docker image to run this pipeline job. We use <a href="https://docs.podman.io/en/latest/" target="_blank" rel="noopener noreferrer">podman</a> as the pipeline job’s base Docker image. Podman (the POD manager) is an open-source tool for developing, managing, and running containers on your Linux systems. It makes creating, managing, and working with Docker images very easy!
  
     ***Note***: ***As some users have encountered errors while using the standard podman images, we have switched to using a custom podman image in the GitLab CI pipeline. Learn more in the [Troubleshooting step](#step13)***.

   * **script** – This is the pipeline job that gets executed as a Docker container (Pod) by the OpenShift Runner instance running on the selected (using tags) OpenShift cluster. There are broadly three tasks done here:

     1. Use podman to build the Docker image for the ppc64le architecture.
     2. Use podman to list the images.
     3. Use podman to push the image to the Quay.io repository. Note the suffix **ppc64le** added to the image tag (also highlighted in the picture below) to denote that it’s a ppc64le architecture

     Learn more about the <a href="https://docs.podman.io/en/latest/markdown/podman-build.1.html" target="_blank" rel="noopener noreferrer">podman build</a> and <a href="https://docs.podman.io/en/latest/markdown/podman-push.1.html" target="_blank" rel="noopener noreferrer">podman push</a> commands.

     ![Figure 39](images/fig39.png)

     <a href="images/fig39.png" target="_blank" rel="noopener noreferrer">View larger image</a>

2. Section 4: **x86-build**

   This section builds the container image (aka Docker image) for the x86 (Intel) architecture. This section also has four components as described above. The only difference being, this will use the x86 OpenShift cluster (hence the **x86** tag used) and create an x86 Docker image, hence the suffix **-x86** is added to the image tag (also highlighted in the picture below).

     ![Figure 40](images/fig40.png)

     <a href="images/fig40.png" target="_blank" rel="noopener noreferrer">View larger image</a>

3. Section 5: **multiarch-manifest**

   This section builds the multi-arch Docker image and pushes it to the Quay.io repository. It has five components as explained below:

   * **stage** – specifies the pipeline stage which this section is part of.
   * **tags** – these are the tags that are used to select which OpenShift cluster will be picked to execute the pipeline job. These tags are used to pick the matching OpenShift runner instance. Here we just specify the **openshift** tag as it doesn’t matter which OpenShift cluster (x86 or ppc64le) is picked to create the multi-arch Docker image.
   * **needs** – This tells the GitLab server to ensure that this job is executed *only after* the x86 and ppc64le build jobs are completed successfully.
   * **image** – specifies the Docker image to run this pipeline job. We use <a href="https://docs.podman.io/en/latest/" target="_blank" rel="noopener noreferrer">podman</a> as the pipeline job’s base Docker image. Podman (the POD manager) is an open-source tool for developing, managing, and running containers on your Linux systems. It makes creating, managing, and working with Docker images very easy!
  
     ***Note***: ***As some users have encountered errors while using the standard podman images, we have switched to using a custom podman image in the GitLab CI pipeline. Learn more in the [Troubleshooting step](#step13)***.

   * **script** – This is the pipeline job that gets executed as a Docker container (aka Pod) by the OpenShift Runner instance running on the selected (using \`tags\`) OpenShift cluster. There are broadly four tasks we do here:

     1. Create a manifest.
     2. Add ppc64le Docker image to the manifest.
     3. Add x86 Docker image to the manifest.
     4. Create and push the multi-arch Docker image to the Quay.io repository, with **-multiarch** as the suffix for the image tag (also highlighted in the picture below).

     Learn more about the <a href="https://docs.podman.io/en/latest/markdown/podman-build.1.html" target="_blank" rel="noopener noreferrer">podman utility</a> & the <a href="https://docs.podman.io/en/latest/markdown/podman-manifest.1.html" target="_blank" rel="noopener noreferrer">podman manifest</a> command.

     ![Figure 41](images/fig41.png)

     <a href="images/fig41.png" target="_blank" rel="noopener noreferrer">View larger image</a>

Congratulations! You now have a basic understanding of how the GitLab CI pipeline works. Let’s execute this pipeline and see how it works!

<div id="step10"></div>

### Step 10. Executing the GitLab CI Pipeline

1. *Prerequisite*: The steps above covered how to setup the GitLab Runner instance for ppc64le architecture cluster and connect it to the GitLab server. Note: Ensure that the same steps have been followed to setup an x86 architecture cluster as well. Do not proceed to the next step unless you have successfully registered both ppc64le and x86 runners with the GitLab server and they are seen under the **Specific runners** section, as shown in your GitLab’s **Settings** -> **CI/CD** -> **Runners**.

     ![Figure 42](images/fig42.png)

     <a href="images/fig42.png" target="_blank" rel="noopener noreferrer">View larger image</a>

     ![Figure 43](images/fig43.png)

     <a href="images/fig43.png" target="_blank" rel="noopener noreferrer">View larger image</a>

1. Now that x86 and ppc64le runners are successfully registered with GitLab server, lets kick off a pipeline run! You can kick off a pipeline run manually or automatically (by changing some code in the repository). To keep things simple, lets kick off a manual run. Click **CI/CD** -> **Pipelines** and click **Run pipeline**.

     ![Figure 44](images/fig44.png)

     ![Figure 45](images/fig45.png)

     <a href="images/fig45.png" target="_blank" rel="noopener noreferrer">View larger image</a>

1. In the resulting **Run pipeline** page, click **Run pipeline**.

     ![Figure 46](images/fig46.png)

1. You will be taken to the **Pipelines** page, where you should see the graphical representation of the pipeline being executed. It will show two (parallel) jobs under the **container-build** step, one each for the ppc64le and x86 case, and one job under **multiarch-push** step for creating the multi-arch Docker image, which is in line with the **.gitlab-ci.yml** file (refer Step 7).

     ![Figure 47](images/fig47.png)

1. You can click on each of the jobs (ppc64le and x86 builds) and witness the execution of the CI job for each architecture. The CI job will build the Docker image for that architecture and push it to the Quay.io registry. Wait for both the architecture-specific jobs to complete.

   Here is what a CI job execution looks like:

     ![Figure 48](images/fig48.png)

     <a href="images/fig48.png" target="_blank" rel="noopener noreferrer">View larger image</a>

     ![Figure 49](images/fig49.png)

     <a href="images/fig49.png" target="_blank" rel="noopener noreferrer">View larger image</a>

     ![Figure 50](images/fig50.png)

     <a href="images/fig50.png" target="_blank" rel="noopener noreferrer">View larger image</a>

     ![Figure 51](images/fig51.png)

     <a href="images/fig51.png" target="_blank" rel="noopener noreferrer">View larger image</a>

1. Once both the jobs under **container-build** is complete, it will then execute the job under **multiarch-push**, which will combine the architecture specific Docker images into a multi-arch Docker image and push that to the Quay.io registry.

   This is what the multi-arch job looks like:

     ![Figure 52](images/fig52.png)

     <a href="images/fig52.png" target="_blank" rel="noopener noreferrer">View larger image</a>

1. Wait for all the pipeline jobs to complete successfully. Under **CI/CD**, click **Pipelines** and then click on the topmost pipeline that corresponds to the most recent pipeline run.

     ![Figure 53](images/fig53.png)

     <a href="images/fig53.png" target="_blank" rel="noopener noreferrer">View larger image</a>

1. Once all the jobs are complete, the pipeline should look as shown. This signifies that your Gitlab CI pipeline executed all jobs successfully!

     ![Figure 54](images/fig54.png)

Congratulations! You have successfully executed your GitLab CI pipeline, thus creating a multi-arch Docker image and pushed it to the Quay.io registry.

<div id="step11"></div>

### Step 11. Verify Docker image creation inQuay.io registry.

1. Navigate to *your* Quay.io repository. Mine is at <a href="https://quay.io/repository/dpkshetty/demos" target="_blank" rel="noopener noreferrer">https://quay.io/repository/dpkshetty/demos</a>.

   Click on the **Tags** icon and you should be able to see three new Docker images created in the recent past, listed on the top of the page. These three Docker images correspond to the x86 architecure, ppc64le architecture and the multi-arch Docker images created by the successful execution of your GitLab CI pipeline.

     ![Figure 55](images/fig55.png)

Congratulations! If you see the above in *your* Quay.io repository, you have successfully managed the integration of GitLab with Quay.io and created the architecture-specific and multi-arch Docker images. Now let’s go ahead and validate that the multi-arch Docker image works as expected.

<div id="step12"></div>

### Step 12. Validate the multi-arch Docker image created in Quay.io.

1. We will validate the multi-arch Docker image by trying to create an application using the same Docker image in both x86 and ppc64le OpenShift clusters. Go to your x86 and ppc64le OpenShift clusters and create a new application using the multi-arch Docker image.

   Note: You can refer to <a href="https://developer.ibm.com/tutorials/build-docker-images-from-github-source-using-openshift-pipelines/#sec7" target="_blank" rel="noopener noreferrer">step 7 of this tutorial</a> for detailed instructions on how to create a new application using a Docker image stored in Quay.io repository.

   Of course, in my case, the Quay.io Docker image URL will be:

   **quay.io/dpkshetty/demos:gitlab-pyflask-multiarch**

   Your URL will be different based on your Quay.io login and repository used.

1. Here’s how the application shows up in my OpenShift clusters.

   On the x86 cluster, I was able to spin up the application successfully using the multi-arch Docker image.

     ![Figure 56](images/fig56.png)

   Similarly, on the ppc64le cluster too, I was able to spin up the application successfully using the multi-arch Docker image.

     ![Figure 57](images/fig57.png)

1. Clicking on the URL under **Routes** in each cluster, displays my application console. It proves that the multi-arch Docker image indeed works as expected! The same image automatically serves the right Docker image based on the hardware architecture of the OpenShift cluster in which the image is being deployed!

   On the x86 cluster:

     ![Figure 58](images/fig58.png)

   On the ppc64le cluster:

     ![Figure 59](images/fig59.png)

Congratulations! You have successfully deployed applications on x86 and ppc64le OpenShift clusters using a multi-arch Docker image.

<div id="step13"></div>

### Step 13. Troubleshooting

1. This tutorial was tested successfully on OpenShift v4.10. Some users have reported seeing the following **uid_map** permission error while running on OpenShift v4.11/v4.12:

   ```
   $ podman --storage-driver=vfs build --isolation chroot -t $APP -f ./Dockerfile --no-cache
   time="2023-04-03T09:21:40Z" level=warning msg="Using rootless single mapping into the namespace. This might break some images. Check /etc/subuid and /etc/subgid for adding sub*ids if not using a network user"
   Error: cannot write uid_map: write /proc/55/uid_map: operation not permitted
   Cleaning up project directory and file based variables
   00:00
   ERROR: Job failed: command terminated with exit code 1
   ```

   * To resolve this issue, we have built a custom multi-arch (x86 and ppc64le) podman image. Refer to this <a href="https://github.com/dpkshetty/podman_gitlab" target="_blank" rel="noopener noreferrer">Git repo </a> for the code and to this <a href="https://quay.io/repository/dpkshetty/podman?tab=tags" target="_blank" rel="noopener noreferrer">Quay.io repository</a> for the podman image.
   * We have also switched to using the custom podman docker image in the GitLab repository referenced in this tutorial, as that works in the old and new OpenShift versions.
   * For the curious minds - the custom podman image takes the <a href="https://quay.io/containers/podman" target="_blank" rel="noopener noreferrer">standard podman image</a> and ensures it runs as **USER podman** and **WORKDIR** is set to **/home/podman**. Please note that the standard podman image available at Quay.io ends up running as root user, which doesn’t play well with the uid/gid mapping available in the standard podman image, resulting in the error above.
   * Thanks to <a href="https://www.linkedin.com/in/federicovagnini/" target="_blank" rel="noopener noreferrer">Federico Vagnini </a> for helping with this solution.

## Summary

This tutorial gives a small glimpse into how GitLab can be used to create a CI pipeline where we can create multi-arch Docker images in a consistent and automated way. GitLab is the DevOps platform that empowers organizations to maximize the overall return on software development by delivering software faster and efficiently.

As an additional exercise, you can try to use the x86 Docker image (**quay.io/dpkshetty/demos:gitlab-pyflask-x86** in my case) to deploy an application in the ppc64le OpenShift cluster and you can see how the Pod errors out, proving that Docker images are hardware architecture-specific! This necessitates the need to build and manage different Docker images for each hardware architecture, which adds complexity to the application lifecycle and Docker image registry management in OpenShift, especially when you have multiple OpenShift clusters in your environment.

Ability to create multi-arch Docker images in an automated way helps ease the whole application and image management in OpenShift.

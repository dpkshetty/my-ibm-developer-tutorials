---
# Retired by TS in LCM backlog; unretired bc part of LP
# MC: Oct 2023: Continues to get really low views;  LP has tons of items in it, so maybe just retire this tutorial and remove it from the LP?
draft: false
ignore_prod: false
display_in_listing: true
title: Deliver webhooks without worrying about firewalls
subtitle: >-
  Achieve DevOps style automation for your application deployment even in restricted environments
meta_title: Deliver webhooks without worrying about firewalls
authors:
  - name: Deepak C Shetty
    email: deepakcshetty@in.ibm.com
  - name: Geoffrey Pascal
    email: geoffrey.pascal@fr.ibm.com
  - name: Sebastien Chabrolles
    email: s.chabrolles@fr.ibm.com
completed_date: '2021-09-17'
last_updated: '2021-09-17'
check_date: "2024-09-20"
excerpt: >-
  Achieve DevOps style automation for your application deployment even in restricted environments
meta_description: >-
  Achieve DevOps style automation for your application deployment even in restricted environments
meta_keywords: >-
  S2I, DevOps, CD, Continuous Deployment, pyflask, OpenShift, github webhooks,
  webhooks, firewall
primary_tag: containers
tags:
  - infrastructure
  - devops
components:
  - redhat-openshift-ibm-cloud
related_links:
  - title: smee.io webhook payload delivery service
    description: >-
      Webhook payload delivery service. Receives payloads then sends them to
      your locally running application.
    url: https://smee.io/
also_found_in:
  - "learningpaths/exploring-openshift-powervs/"
---

## Introduction

Red Hat OpenShift Source-to-Image (S2I) is a framework that makes it easy to write images by taking application source code as input and producing a new image and then running the assembled application as output. The main advantage of using S2I for building reproducible container images is the ease of use it provides for developers.

Developers mostly use GitHub to maintain version control of their source code. Webhooks is a GitHub feature to notify an external server or an entity about the events that occur in the GitHub repository, so that the server or the entity can take further suitable action for the notified event. Refer to the [Enable continuous deployment using Red Hat OpenShift S2I and GitHub webhooks](/tutorials/continuous-deployment-s2i-and-webhooks/) tutorial to learn  how GitHub webhooks can help in achieving continuous deployment in association with OpenShift S2I feature.

One of the persistent issues that Enterprise developers face is how to make GitHub webhooks work when an OpenShift cluster is behind a firewall (typically behind a company’s firewall), as GitHub webhooks cannot pass through the firewall, and thus, breaking the  continuous deployment workflow.

In this tutorial, you’ll learn how to use the [smee.io](https://smee.io/) web-based webhook payload delivery service to make GitHub webhooks work through a firewall. This tutorial explains the procedure to install the smee client on the OpenShift cluster, connect the client to the smee.io web channel at one end and your OpenShift application webhook endpoint at the other end, and showcase an end-to-end scenario of an automated continuous deployment.

##### Figure 1. High-level view of the solution architecture

![figure 1](https://github.com/user-attachments/assets/b78bf617-5c6a-4032-9095-6bfb2a62a0d2)


## Prerequisites

To make GitHub webhooks work through a firewall using smee.io, you need to make sure that the following prerequisites are fulfilled:

* An OpenShift cluster (any platform or architecture and version, preferably behind firewall) In this tutorial, we are using OpenShift Container Platform v4.8.x, running behind the company’s firewall
* Familiarity with GitHub basic operations (for example, clone and create repo, create and update files in repo, and so on)

## Estimated time

It would take around 45 minutes to use the smee.io web-based webhook payload delivery service to make GitHub webhooks work through a firewall.

## Steps

1. [Set up a DevOps workflow using OpenShift S2I feature and GitHub](#step1)
2. [Set up GitHub webhooks for continuous deployment – It fails!](#step2)
3. [Set up smee.io as a proxy for GitHub webhooks](#step3)
4. [Test continuous deployment workflow – It works!](#step4)

<div id='step1'><div>

### 1. Set up a DevOps workflow using OpenShift S2I feature and GitHub

   <div id='step1sub1'><div>

1. Clone the required GitHub repository.

   We have created the following GitHub repo and hosted a very simple pyflask app, which has three API endpoints (root, version, and test).

   [https://github.com/ocp-power-demos/s2i-pyflask-demo](https://github.com/ocp-power-demos/s2i-pyflask-demo)

   Fork this repository because you will be making changes to the GitHub code as part of this tutorial which will need GitHub owner privileges.

   The clone we created for the repository is at:  
   [https://github.com/dpkshetty/s2i-pyflask-demo](https://github.com/dpkshetty/s2i-pyflask-demo)

   **Note**: Your GitHub URL would be different.

1. Create a project (namespace) to host pods for this use case.

   You can skip this step if you don’t have permissions to create a new project or have been provided with a pre-created project/namespace.

   This is under the assumption that you have permission to create a new,  project. In the **Administrator** persona, click **Home > Projects**. On the Projects page, click **Create Project**.

   ![image](https://github.com/user-attachments/assets/d30ca182-1080-4a07-af61-2a9babb6da9b)

   On the Create Project page, enter **smee-demo** as the name of the project and click **Create**.

   ![image](https://github.com/user-attachments/assets/889e9d7b-71c6-411b-b6ba-efe19fc5f64e)

1. Switch to the **Developer** persona and click **Topology**. Ensure that you are in the right project (the one you just created or pre-assigned to you).

   ![image](https://github.com/user-attachments/assets/e3eb57bf-b19c-44bf-8431-59a029187afc)

1. Click **+Add** and then click **From Git**.

   ![image](https://github.com/user-attachments/assets/91100ec8-94ca-461a-afd9-b6cb7d7ae99f)

1. In the resulting form, enter `https://github.com/dpkshetty/s2i-pyflask-demo` in the Git Repo URL field and then click to expand **Show Advanced Git Options**.

   **Note**: Use your cloned GitHub URL (see [step 1](#step1sub1)).

   Then, click **Show Advanced Git Options** and enter **main** in the Git reference field (as we are using the main Git branch of the repo in this example).

   ![image](https://github.com/user-attachments/assets/9b0ff021-bc15-4422-8345-de897e3f9550)

1. In the Builder section, click **Python**.

   ![image](https://github.com/user-attachments/assets/1a4fe2ae-f9d9-4bc8-9e34-2dea6d8838cf)

1. In the General section, enter the application name as **smee-demo** and the name as **smee-demo**. Then, in the Resources section, select **Deployment**.

   ![image](https://github.com/user-attachments/assets/61d0a1d5-ea83-47a5-8c9b-539a4c658cd9)

1. In the Advanced options section, select the **Create a route to the Application** checkbox and click **Create**.

   ![image](https://github.com/user-attachments/assets/a26ba958-e673-4662-9fa5-b9f9f9d17b6d)

1. Notice that you are in the **Topology** view where you can see an icon for your application you just deployed. Click **smee-demo** and then click the **Resources** tab on the right side and wait for the build to complete and the pod to be in the **Running** state.

   ![image](https://github.com/user-attachments/assets/053513f5-070b-40db-b1f1-aae31a97453b)

1. In the Routes section, click the URL. It opens in a new browser window, and you should be able to see the **Hello, World ! - Pyflask Demo** message from the application you just deployed.

   **Note**: The exact URL in your case may vary as it is OpenShift cluster set up dependent.

   ![image](https://github.com/user-attachments/assets/78a983b3-6ef4-48ec-ad0f-490ba84190ec)

   ![image](https://github.com/user-attachments/assets/594ac4cf-aa94-4df5-b9e1-bf31b764d1e3)

1. In the browser, append **/version** at the end of the existing URL to access the version API endpoint and ensure you are seeing the message, **App version : 1.0**.

   ![image](https://github.com/user-attachments/assets/943c25f5-d856-41a1-9305-9121f5d09294)

   You have successfully used OpenShift S2I and the GitHub repository to deploy your application.

   Now, let’s set up GitHub webhooks to automate application deployment.

   <div id='step2'><div>

### 2. Set up GitHub webhooks for continuous deployment – It fails!

GitHub webhooks allow external services to be notified when certain GitHub events happen.

Continuous deployment means that updates to the GitHub source code must automatically deploy new pods. For this to work, we need to configure our GitHub repo with the webhook that OpenShift provides as part of the *BuildConfig* attribute.

1. On the **Resources** tab, click BuildConfig (BC) entry **smee-demo**.

   ![image](https://github.com/user-attachments/assets/f40fbf20-477f-433d-a7bc-e6496957ed9a)

   <div id='step2sub2'><div>

1. On the BuildConfig details page, scroll down and in the Webhooks section, and in the row corresponding to the GitHub entry, click **Copy URL with Secret**. Save this URL as you need to refer to this in a later step.

   ![image](https://github.com/user-attachments/assets/daee742a-8a90-4008-8a4f-7d324c9336f3)

   ![image](https://github.com/user-attachments/assets/c07f6ad8-6359-401b-a920-06e1df24863d)

1. Switch to your browser and navigate to your GitHub repo. Click **Settings** and then click the **Webhooks** tab. On the Webhooks page, click **Add webhook**.

   ![image](https://github.com/user-attachments/assets/70f1f717-1287-4547-9d72-d6e19480415b)

1. In the resulting form, enter the following details:

   * **Payload URL**: `<`Github webhook URL as copied from [step 2](#step2sub2)`>`
   * **Content type**: application/json
   * Retain the default values for the remaining fields and click **Add webhook**.

   ![image](https://github.com/user-attachments/assets/d1958b2a-94ba-4ee5-b2ff-951854bcdbcf)

1. If successful, notice that your newly added webhook is displayed on the Webhooks page. Click the newly created **webhook** to view the webhook details page.

   ![image](https://github.com/user-attachments/assets/0ee27a28-66f7-403e-bcbe-9e2f3667e12b)

1. GitHub generally does a simple ping test as part of adding a new webhook. On the webhook details page, click the **Recent Deliveries** tab and notice that an entry prefixed with an exclamation mark (indicating that the ping test has failed) is displayed. Click the entry to find more details about the REST API call and the associated response for the ping test and you can see the failure reason as **timed out**.

   ![image](https://github.com/user-attachments/assets/d1a670e7-ca5e-44e1-b9fa-bba4a793b0b9)

1. This is expected because the OpenShift cluster used in this example is behind company firewall, and therefore, the OpenShift API server hosting the webhook URL endpoint is not reachable by github.com.

   Next, let’s see how we can use the smee.io web-based webhook payload service to get past this constraint.

<div id='step3'><div>

### 3. Set up smee.io as a proxy for GitHub webhooks

1. In your browser, navigate to [https://smee.io/](https://smee.io/) and click **Start a new channel**.

   ![image](https://github.com/user-attachments/assets/96756809-e35e-457b-9138-893236ac927d)

   <div id='step3sub2'><div>

1. On the resulting page, copy the URL in the Webhook Proxy URL field and save it somewhere safe so that you can recover it, if required and in case this page gets lost.

   **Note**: Your URL will be unique and different.

   ![image](https://github.com/user-attachments/assets/55133439-0702-402e-86d6-726348ba9ece)

1. Go back to GitHub Webhooks page, click your webhook and update the URL in the **Payload URL** field with the proxy URL you copied in the previous step. Retain the existing values in the remaining options and click **Update webhook**.

   ![image](https://github.com/user-attachments/assets/6d2f80ab-8337-4248-a9b3-1835e8cd0122)

   ![image](https://github.com/user-attachments/assets/33150429-1d70-4812-a06d-1804489d42c1)

1. We have now updated GitHub asking it to deliver the webhook to the smee.io webhook proxy URL. Click on **Recent Deliveries**, then click the ellipsis button (with three dots) and click **Redeliver**.

   ![image](https://github.com/user-attachments/assets/0f8908a9-978b-4236-a089-62927f224f01)

1. Notice a second entry under **Recent Deliveries** with a **tick mark** associated (indicating that it is successful).

   ![image](https://github.com/user-attachments/assets/a53abaf1-8dc9-454a-b414-1a4eee0a18a1)

1. In your browser window, navigate to the smee.io page and observe that it indeed has **received the ping** from GitHub. With this, we have reached half-way with the webhook reaching smee.io.

   ![image](https://github.com/user-attachments/assets/8ddd5ac2-89f1-40a1-9405-f3ac8a5fa714)

1. The next task is to deploy the smee-client application on our OpenShift cluster using a pre-existing container image. In  your browser, navigate to your OpenShift cluster console page and ensure that you are in the **Developer** persona. Select the required project and, click **+Add**. Then, click **Container images**.

   ![image](https://github.com/user-attachments/assets/b60930e2-4e74-4c00-82e0-1e6a223c80e8)

1. In the resulting form, enter the following values for the corresponding fields:

   **Image name from external registry**: quay.io/schabrolles/smeeclient:stable

   **Name**: smee-client

   Retain the default values for the remaining fields or options and click **Create**.

   ![image](https://github.com/user-attachments/assets/102cc17d-9f34-4bdc-8697-93789fa9053f)

   ![image](https://github.com/user-attachments/assets/d1b7b5d5-f20c-460a-af23-0ea8b9c1c2fc)

   ![image](https://github.com/user-attachments/assets/8a2507b7-e381-4050-af75-28e693ea560f)

1. In the Topology view, click the **smee-client** application icon and on the Resources tab (after some time) notice the message **CrashLoop BackOff** indicating that the pod fails to start. This is expected as we still haven’t configured the smee-client application.

   ![image](https://github.com/user-attachments/assets/7a230c6d-27ad-45af-8ce8-1f3d5682e316)

1. Let’s configure the smee-client with the right environment variables. smee-client requires two environment variables (SMEESOURCE and HTTPTARGET) to be set up. You can find more details about smee-client on the [GitHub page](https://github.com/geoffrey-pascal/smeeclient-nodejs). Click **smee-client** on the right pane.

   ![image](https://github.com/user-attachments/assets/2515a9cc-b650-49ba-93e4-93418b86a307)

1. On the **smee-client** deployment page, enter values for the following two environment variables:

   **SMEESOURCE**: [https://smee.io/640vAKQUe9pmpPfH](https://smee.io/640vAKQUe9pmpPfH)

   **Note**: Your URL will be different. See [step 3](#step3sub2).

   **HTTPTARGET**:  
   [https://api.p1255.cecc.ihost.com:6443/apis/build.openshift.io/v1/namespaces/smee-demo/buildconfigs/smee-demo/webhooks/928df75e56d0f617/github](https://api.p1255.cecc.ihost.com:6443/apis/build.openshift.io/v1/namespaces/smee-demo/buildconfigs/smee-demo/webhooks/928df75e56d0f617/github)

   **Note**: Your URL will be different. See [step 2](#step2sub2).

   If required, click **Add more** to add a new environment variable and click **Save**.

   ![image](https://github.com/user-attachments/assets/fb9c9c53-c826-4c31-a66c-538df068a875)

   ![image](https://github.com/user-attachments/assets/cff33685-aa17-46ae-bc5e-bc240aecbe2f)

1. The smee-client application will get redeployed as its configuration was changed. In the **Topology** view, click the **Resources** tab, and wait for the smee-client pod to show the status as **Running**.

   ![image](https://github.com/user-attachments/assets/f45fd8f0-85ed-46c7-b209-8a18cd6e11f2)

1. Click **View logs** to see the logs for the smee-client application. It should state that its now forwarding all the smee.io channel’s payload to the GitHub webhook endpoint URL.

   ![image](https://github.com/user-attachments/assets/8638a8d6-34d9-4dc6-bdb3-964d654f4e6d)

   ![image](https://github.com/user-attachments/assets/3844ba47-7696-4ec8-b127-17b21b37be66)

   You have now successfully set up the smee-client application to pull the webhook payload from the smee.io channel and deliver it to your OpenShift application’s webhook endpoint URL. Now, let’s test the continuous deployment workflow!

<div id='step4'><div>

### 4. Test continuous deployment workflow – It works!

As part of this step, we will make minor updates to the code in our GitHub repository and check if smee.io and smee-client are collectively able to deliver the webhook to OpenShift and if our application is auto-built and auto-deployed.

1. In your browser, navigate to your GitHub repository page and click the **app.py** file.

   **Note**: Your GitHub URL will be different

   ![image](https://github.com/user-attachments/assets/aac3656a-97a0-4564-8a53-487d2d6566da)

1. Click the pencil icon, change the version from **1.0** to **2.0**, add a commit message, and click **Commit changes**.

   ![image](https://github.com/user-attachments/assets/44a42c2b-1be6-42ff-9e9a-7fdd084b9bb0)

   ![image](https://github.com/user-attachments/assets/94626378-327d-41fe-af62-c7136275d847)

   ![image](https://github.com/user-attachments/assets/a7f0b221-ce3d-4df8-830d-d9b53df34bfe)

   <div id='step4sub3'><div>

1. In your browser, navigate to the OpenShift console. On the Topology page, on the **Resources** tab, notice that the new build for **smee-demo** has automatically started.

   ![image](https://github.com/user-attachments/assets/828f6ce2-a7d5-4316-9244-57b671645850)

1. A new version of our application will be auto-deployed after the build completes. Wait for the build to complete and ensure that the new pod/container shows the status as **Running**.

   **Note**: The name of the new pod/container is now different (compare with [step 4](#step4sub3)).

   ![image](https://github.com/user-attachments/assets/9eb23e86-4dce-49f5-9b3d-87635337743b)

1. (Optional) In your browser, navigate to Github webhooks page and verify that a successful notification is sent for the code change event.

   ![image](https://github.com/user-attachments/assets/61533687-d030-477e-a7ab-6c2e9b384b74)

1. (Optional) Go to the smee.io page and verify that it shows a new **push** event for the recent code change and commit operations we did in the GitHub repository.

   ![image](https://github.com/user-attachments/assets/393d495e-4c88-4e40-bf12-4687c6626313)

1. (Optional) Go to OpenShift console, and in the **Topology** view, for the smee-client application, open the **Resources** tab and click **View Logs**. You can see that smee-client indeed sent a **POST https** message to our GitHub webhook endpoint URL, due to which OpenShift rebuilt and redeployed our application automatically!

   ![image](https://github.com/user-attachments/assets/2b2c3d66-9ccb-49b0-9674-261842e3b489)

   ![image](https://github.com/user-attachments/assets/875a5fe1-c3c4-4ef9-980d-c9ec61bfc62b)

1. Let’s verify that the application running is indeed the newer version. In the OpenShift console click **Topology**. For the **smee-demo** application, on the Resources tab, click the URL in the Routes section.

   ![image](https://github.com/user-attachments/assets/a13aaadd-090b-465a-bc8e-f843a3ec7b0a)

1. The resulting page in the browser should show the message, **Hello, World ! - Pyflask Demo** indicating that the new version of our application is running successfully.

   ![image](https://github.com/user-attachments/assets/3af31ad7-9b2b-42d0-815b-6b43df690a38)

1. Access the **/version** endpoint of our application and it should show **App version : 2.0** which proves that our recent code change in GitHub indeed resulted in a new application being auto-built and auto-deployed!

   ![image](https://github.com/user-attachments/assets/f19032cb-e97d-417c-800e-c6ef8cd6796d)

## Summary

Congratulations! You have successfully demonstrated continuous deployment using OpenShift S2I and GitHub webhooks, in spite of the OpenShift cluster being located behind the company’s firewall. This tutorial shows how to use the smee.io web service to act as a proxy for the GitHub webhook and use the smee-client application in our OpenShift cluster to forward the webhook to an OpenShift API server running behind a firewall. smee.io is intended for use in development, and not for production. You can find more details in the [GitHub repository](https://github.com/probot/smee.io#usage).

Nevertheless, it is a great tool for developers to stop worrying about firewall issues and focus on application development.

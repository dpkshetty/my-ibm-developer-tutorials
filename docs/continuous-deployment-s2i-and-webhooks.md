---
# Related publishing issue: https://github.ibm.com/IBMCode/IBMCodeContent/issues/5721
# https://w3.ibm.com/developer/contentcreatortool/cct/developer/master/Tutorials/
draft: false
ignore_prod: false
completed_date: "2021-02-17"
check_date: "2024-12-17"
last_updated: "2022-04-06"
excerpt: Use OpenShift S2I feature to build and deploy a cloud-native application directly from github source and use github webhook to automate application deployment for new code changes and achieve DevOps style Continuous Deployment
tags:
  - infrastructure
  - linux
  - continuous-delivery
  - devops
  - ci-cd
components:
  - redhat-openshift-ibm-cloud
abstract: Use OpenShift S2I feature to build and deploy a cloud-native application directly from github source and use github webhook to automate application deployment for new code changes and achieve DevOps style Continuous Deployment
authors:
  - name: Deepak C Shetty
    email: "deepakcshetty@in.ibm.com"
  - name: Pradipta Banerjee
    email: "bpradipt@in.ibm.com"

meta_keywords: S2I, DevOps, CD, Continuous Deployment, pyFlask, OpenShift
primary_tag: "linux-on-ibm-power"
title: Enable continuous deployment using Red Hat OpenShift S2I and GitHub webhooks
subtitle: Achieve DevOps-style automation for your application
also_found_in:
  - "learningpaths/exploring-openshift-powervs/"
---

## Introduction

Red Hat® OpenShift® Source-to-Image (S2I) is a framework that makes it easy to write images that take application source code as input and produce a new image that runs the assembled application as output. The main advantage of using S2I for building reproducible container images is the ease of use for developers.

In this tutorial, you&#39;ll learn how to use the OpenShift S2I feature with a cluster deployed on IBM® Power Systems™ Virtual Servers. We&#39;ll use a sample pyFlask (Python-based popular web framework) application hosted on GitHub. Additionally, you'll learn how to set up GitHub webhook to notify OpenShift of new code push/commit events in GitHub, such that OpenShift will auto-rebuild and auto-redeploy the application using the latest code changes in your GitHub repository.

## Prerequisites

To enable continuous deployment using the OpenShift S2I feature, make sure that the following prerequisites are fulfilled:

* Access to the OpenShift cluster in Power Virtual Server (we used OpenShift Container Platform version 4.6.9)
* Familiarity with GitHub basic operations (clone/create repo, create and update files in repo, and so on)

## Estimated time

To enable continuous deployment using the OpenShift S2I feature as mentioned in this tutorial takes around 30 minutes.

## Steps

1. [Create an OpenShift project to deploy the application](#st1)
1. [Clone the GitHub repo](#st2)
1. [Create a new deployment from your GitHub repo](#st3)
1. [Verify that the pyFlask app is running successfully](#st4)
1. [Set up GitHub webhook to notify OpenShift of GitHub events and set up auto deployment](#st5)
1. [Change source code in GitHub and verify that pod is auto deployed](#st6)

<a name="st1" />

### 1. Create an OpenShift project to deploy the application.

1. Click the **Projects** tab and on the Projects page, click **Create Project**.

   ![image](https://github.com/user-attachments/assets/ad64db74-bf1e-44f4-a5c6-a23cba4eb61c)

   ![image](https://github.com/user-attachments/assets/da3a0fa3-4578-4172-a89e-8b631f80ab73)

1. In the Create Project dialog box, enter project details such as name, display name, and description and click **Create**.

1. On the Project page, verify that the project is created.

   **Tip**: If there are multiple projects, you can use filters to quickly search for the project.

   ![image](https://github.com/user-attachments/assets/8e041436-8f6b-49e8-9d4b-079f5b6b795a)

<a name="st2" />

### 2. Clone the GitHub repo.

A simple pyFlask app with three API endpoints (root, version, and test) is hosted in the following link:

[https://github.com/ocp-power-demos/s2i-pyflask-demo](https://github.com/ocp-power-demos/s2i-pyflask-demo)

You need to fork this repository because you will be making changes to the GitHub code as part of this use case which will need owner privileges.

<a name="st3" />

### 3. Create a new deployment from your GitHub repo.

1. In the OpenShift GUI, switch to the Developer persona and click **+Add** to create a new deployment. Then, click the **From Git** option. Ensure that you have selected the **pyflask-demo** project we created earlier for this demo.

   ![image](https://github.com/user-attachments/assets/5b9cb9b4-1722-40e7-aa65-4a9dadd0a947)

1. Fill the **Import from Git** form with the required details.

   **Note**: Do not forget to add your GitHub repo path in the respective field.

   ![image](https://github.com/user-attachments/assets/fbc3ebff-f320-4d6e-bb74-a9b270b05fed)

1. Ignore the **Unable to detect builder image** message if it appears.

1. Because OpenShift failed to auto-detect the source code language, we need to select it manually. Click the **Python** builder image.

   ![image](https://github.com/user-attachments/assets/5c898d11-fe80-4ca9-989a-c8e39850fb4b)

1. Enter a unique name for OpenShift to use for the application and the resources it will create as part of this deployment.

   ![image](https://github.com/user-attachments/assets/7ac97026-3553-49c2-874e-078363d41525)

1. OpenShift in Power Virtual Server by default only supports secure (HTTPS) routes. So you will need to make some config changes. Click **Routing**, and in the Routing form, select the **Secure Route** checkbox to enable TLS edge termination.

   ![image](https://github.com/user-attachments/assets/ad853f04-ddfd-4e55-80f4-877891e9bfcf)

   ![image](https://github.com/user-attachments/assets/ba9ca85d-221d-4807-b291-3e1054bda4cb)

   ![image](https://github.com/user-attachments/assets/dedddec1-900d-4b8f-ae93-d79af1c32d2e)

1. Retain the default values for the remaining fields and click **Create**.

1. In the Topology view, click your application. In the Deployment view, wait for the build to complete and the pod to be in **Running** state.

   What happens here is that OpenShift builds the container image (using the Python builder image and your source code from your GitHub repo), creates a Docker image, uploads the image to the OpenShift internal image registry, and creates a pod using that Docker image.

   ![image](https://github.com/user-attachments/assets/ccc53566-864c-44c4-849c-74fbe510c5d7)

<a name="st4" />

### 4. Verify that the pyFlask app is running successfully.

1. In the Topology view, click the **s2i-pyflask** application, scroll down to the bottom of the resources page, and click the Routes URL.

   ![image](https://github.com/user-attachments/assets/a7e16326-cd96-4d5d-83d4-b3316f2a6b55)

   The URL opens in your browser and displays the &quot;Hello World….&quot; message as mentioned in the pyFlask application GitHub source code (refer to the `app.py's hello_world()` function which maps to the root endpoint).

   **Note**: While opening this URL for the first time, if you encounter a browser security warning, click **Advanced** and continue to open the URL.

   ![image](https://github.com/user-attachments/assets/9e1b0e90-95a7-4795-95b0-afb209a41cd2)

1. Similarly, open the /version and /test endpoints, and notice that the appropriate message as encoded in the application's GitHub source code is displayed.

   ![image](https://github.com/user-attachments/assets/c1ecb28a-4c68-4d9d-b36d-5bce08f85ae6)

   ![image](https://github.com/user-attachments/assets/80e878b1-3892-4135-bb0a-727f32f110e4)

   This proves that the pyFlask application that was deployed from the GitHub source is working as expected.

   You have now successfully deployed a pod directly from your GitHub repo.

<a name="st5" />

### 5. Set up GitHub webhook to notify OpenShift of GitHub events and set up auto deployment.

Github webhooks allow external services to be notified when certain GitHub events happen.

Continuous deployment means updates to the GitHub source code must automatically deploy new pods. For this to work, we need to configure our GitHub repo with the webhook that OpenShift provides as part of the BuildConfig attribute.

This is best achieved using the GUI.

1. On the BuildConfigs page, click your buildconfig entry and scroll down to find the Generic webhook and click **Copy URL with Secret**.

   ![image](https://github.com/user-attachments/assets/ba7e0895-66b7-47d6-aa95-5bdf6343eb78)

   ![image](https://github.com/user-attachments/assets/5bb4c78a-75ee-4b7f-84c9-f0129c1a69b8)

1. Navigate to your GitHub repo, and under Settings, click the **Webhooks** tab and add a new webhook using the copied URL from the previous step.

   ![image](https://github.com/user-attachments/assets/78c8e515-9982-4513-afb1-bdad5d3a8b79)

1. Enter the copied URL in the **Payload URL** field, select the **application/json** option from the **Content type** list, and click **Disable (not recommended)** for SSL verification. Retain the default values for the remaining fields and click **Add webhook**.

   ![image](https://github.com/user-attachments/assets/38b9c31d-0225-4871-8b5a-3ca0a3bb3c97)

1. If successful, notice that your newly added webhook is displayed on the Webhooks page.

   ![image](https://github.com/user-attachments/assets/18e9ff6b-5489-44b0-bdc5-73292e7dc308)

1. Click your newly added webhook to see details. GitHub generally does a simple ping test as part of adding a new webhook. Scroll down to the Recent Deliveries section and notice that an entry prefixed with a tick mark (indicating that the ping test is successful) is displayed. Click the entry to find more details about the REST API call and the associated response for the ping test.

   ![image](https://github.com/user-attachments/assets/5c8e8dc4-1f73-4042-851d-169572e17b00)

   A successful ping test (Response 200) would mean that GitHub is able to connect with your OpenShift cluster.

   ![image](https://github.com/user-attachments/assets/1e6abdb8-7a07-471f-9eee-8e5dc54376da)

   You have now successfully set up the GitHub webhook.

<a name="st6" />

### 6. Change source code in GitHub and verify that pod is auto deployed.

Let&#39;s now make a small change in source code by changing the version to 2.0 in app.py. This should trigger a push event from GitHub to the OpenShift and in turn causes OpenShift to rebuild the Docker image and redeploy our pod using the newly built Docker image.

1. Open the app.py file and in the `get_version()` function, change the version from `1.0` to `2.0`.

   ![image](https://github.com/user-attachments/assets/98f05263-51f4-4114-ae26-4931411c7535)

   ![image](https://github.com/user-attachments/assets/fb7750da-385b-4427-b133-4859cef0de27)

1. Scroll down, add a commit message, and click **Commit changes**.

   ![image](https://github.com/user-attachments/assets/9a25646f-a98e-495e-985d-02f80b7f85c5)

1. Then, switch to your OpenShift GUI, switch to the Developer persona/view, click the **s2i-pyflask** application, and watch a new build getting triggered automatically.

   ![image](https://github.com/user-attachments/assets/b98754ec-fcf7-46f1-b407-492504a58941)

   After this build is complete, a new Pod will be auto deployed using the new Docker image that got created from this build.

1. Switch to the Administrator persona/view, click **Workloads**, and then click **Pods.** Notice that the old pod is being terminated and new pod is being created.

   ![image](https://github.com/user-attachments/assets/8b5dafd7-fd5b-4211-affa-f06f9e05625f)

   ![image](https://github.com/user-attachments/assets/adec66ff-83cd-49cf-a987-81517458ffb8)

1. Switch back to the Developer persona/view, click **Topology**, select the **s2i-pyflask** application to open the resources window, and scroll down to the Routes section.

   ![image](https://github.com/user-attachments/assets/a88c0193-f4eb-4674-beee-2b801aee236f)

1. Click route URL and access the /version endpoint.

   Note that the router URL is unchanged while we moved from version 1 to version 2 of our application. So, if you already have the /version endpoint URL opened (as part of the earlier step), you can just refresh it.

   ![image](https://github.com/user-attachments/assets/df5a514b-e905-4678-990f-c0b2c0dcce7e)

   Notice that the container is indeed running the newer version of the application!

   If you go to your GitHub webhooks view, you can see a new entry under Recent Deliveries, which maps to the recent notification delivered by GitHub to OpenShift in response to the code change (push/commit) operation (version change) we did.

   ![image](https://github.com/user-attachments/assets/87c42fcb-e3eb-4903-a182-b2537c08a649)

1. Click the recent delivery and notice the Response 200 message indicating successful rebuild and redeployment of the pod in our OpenShift cluster.

   ![image](https://github.com/user-attachments/assets/1e558c45-99ea-4422-8011-1166b440417c)

## Summary

Congratulations, you have successfully used OpenShift S2I capability on an OpenShift cluster running in Power Virtual Server. You learned how to deploy an application in an OpenShift cluster directly from GitHub-hosted source code, set up the connection between OpenShift and GitHub using webhooks, and ensure that the new code changes to source code repository results in a new version of the application being deployed automatically in an OpenShift cluster. As a next step, I invite you to read this tutorial, <a href="/tutorials/deliver-your-webhooks-without-worrying-about-firewalls/" target="_blank" rel="noopener noreferrer">Deliver webhooks without worrying about firewalls</a> to learn how you can achieve DevOps style automation for your application deployment even in restricted environments.

## Take the next step

Join the <a href="https://community.ibm.com/community/user/powerdeveloper/home" target="_blank" rel="noopener noreferrer">Power Developer eXchange Community (PDeX)</a>. PDeX is a place for anyone interested in developing open source apps on IBM Power. Whether you're new to Power or a seasoned expert, we invite you to join and begin exchanging ideas, sharing experiences, and collaborating with other members today!

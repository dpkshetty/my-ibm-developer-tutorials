---
# Related publishing issue: https://github.ibm.com/IBMCode/IBMCodeContent/issues/5288
# https://w3.ibm.com/developer/contentcreatortool/cct/developer/master/Tutorials/

draft: false

title: "Use Red Hat OpenShift Source-to-Image to deploy applications from Dockerfiles"
subtitle: "Set up GitHub webhooks to trigger new pod deployments"
meta_title: "Use Red Hat OpenShift Source-to-Image to deploy applications from Dockerfiles"

authors:
  - name: "Deepak C Shetty"
    email: "deepakcshetty@in.ibm.com"
  - name: "Masa Abushamleh"
    email: "Masa.Abushamleh@ibm.com"
editors:
  - name: Michelle Corbin
    email: corbinm@us.ibm.com

completed_date: "2021-08-04"
last_updated: "2024-02-27"
check_date: "2025-02-15"

time_to_read: 30

excerpt: "Set up GitHub webhooks to trigger new pod deployments"
meta_description: "Learn how to use Red Hat OpenShift Source-to-Image to deploy applications from Dockerfiles"
meta_keywords: "openshift source-to-image, openshift S2I"

primary_tag: containers
components:
  - docker
  - openshift
  - redhat-openshift-ibm-cloud

---

The Red Hat OpenShift Source-to-Image (S2I) framework eases your ability to create reproducible Docker container images from application source code. A Dockerfile is a recipe (or blueprint) for building Docker images. In this tutorial, you learn how to use OpenShift S2I to build a Docker image from a Dockerfile hosted in GitHub and deploy a container pod by using the image. You also learn how to set up a GitHub webhook to notify OpenShift of new code push and commit events so it automatically rebuilds and redeploys pods by using the latest changes to your code and the Dockerfile within your GitHub repo.

## Prerequisites

For this tutorial, you must have a <a href="https://cloud.ibm.com/kubernetes/catalog/create?platformType=openshift&cm_sp=ibmdev-_-developer-_-trial" target="_blank" rel="noopener noreferrer">Red Hat OpenShift on IBM Cloud</a> cluster (4.3 or higher) and a <a href="https://github.com/pricing" target="_blank" rel="noopener noreferrer">GitHub</a> account.

## Estimated time

This tutorial takes 30 minutes to complete.

## Steps

1. [Fork the GitHub repo and host a Dockerfile](#1-fork-the-github-repo-and-host-a-dockerfile)
1. [Create a project](#2-create-a-project)
1. [Create a pod deployment using the Dockerfile from your GitHub repo](#3-create-a-pod-deployment-using-the-dockerfile-from-your-github-repo)
1. [Verify the container process matches the command specified in the Dockerfile](#4-verify-the-container-process-matches-the-command-specified-in-the-dockerfile)
1. [Set up a GitHub Webhook](#5-set-up-a-github-webhook)
1. [Make changes to your GitHub repository](#6-make-changes-to-your-github-repository)

### 1. Fork the GitHub repo and host a Dockerfile

To fork the <a href="https://github.com/IBM/oc-docker-s2i ">GitHub repository</a> for this tutorial, click the __Fork__ button that is located on the main repo page. Within the repo is a Dockerfile that contains the following lines of code.

```
# simple dockerfile to test OCP s2i using Dockerfile

FROM ubuntu:18.04
CMD ["/bin/bash", "-c", "sleep infinity"]
# CMD ["/bin/bash", "-c", "--", "while true; do sleep 30; done;"]
```

During a later step of this tutorial, you make changes in the Dockerfile to trigger new pod deployments.

### 2. Create a project

* From the Administrator perspective of OpenShift, go to __Projects__ and click __Create Project__.
* Name the project `s2i-project` and click __Create__.
* After the project is created, you are redirected to the Overview tab of the Project Details page.

  ![image](https://github.com/user-attachments/assets/2101e36c-e378-4975-9bc6-7c263e4970be)

### 3. Create a pod deployment using the Dockerfile from your GitHub repo

* In the OpenShift web console, switch from the Administrator perspective to the Developer perspective.

* Select __Topology__ from the navigation panel.

* To create an application, select the __From Dockerfile__ option.

  ![image](https://github.com/user-attachments/assets/bc1c0775-acbd-4ac7-987f-4ae6f4eca619)

* Copy the URL of your GitHub repo, which should be `https://github.com/<GITHUB-USERNAME>/oc-docker-s2i`. Ensure that the URL includes _your GitHub username_ and not the `<GITHUB-USERNAME>` placeholder.

* Paste the URL of your GitHub repository in the __Git Repo URL__ field.

* In the __Context Dir__ field, type `/ubuntu`, which is where the Dockerfile is located.

  ![image](https://github.com/user-attachments/assets/52832ea7-ccb7-4350-91e6-89164ece0cb1)

* In the __Resources__ section, select `Deployment Config` and keep the __Create a route to the application__ option selected.

  ![image](https://github.com/user-attachments/assets/2face5cc-4d7a-46ba-a051-54813c093d08)

* Click __Create__.

* You are redirected to the Topology view, which lists the created pods.

* If you click on your application, you open the Deployment Config (DC) view, where you can review the Details, Resources, and Monitoring logs. What happens is that OpenShift builds the Docker image by using the Dockerfile from your GitHub repo, creates a Docker image, uploads the image into the OpenShift internal image registry, and creates a pod by using that Docker image.

  ![image](https://github.com/user-attachments/assets/adea888e-1673-49da-b2b8-e14f4178cbd9)

### 4. Verify the container process matches the command specified in the Dockerfile

* From the Topology view, click the `DeploymentConfig` icon for your application to open the Deployment Config Details page.

  ![image](https://github.com/user-attachments/assets/d66d7a4c-a248-4cc0-897e-a68ae7f48618)

* Select the __Pods__ tab on the Deployment Config Details page to list the running pod.

  ![image](https://github.com/user-attachments/assets/e8602bf7-5af4-4439-ad65-8f18e76eb2f4)

* Select the running pod to open the Pod Details view.

  ![image](https://github.com/user-attachments/assets/e9b82059-83c9-4248-8c91-acaab400b36d)

* Select __Terminal__, which brings up a terminal (shell) inside your running Ubuntu container.

  ![image](https://github.com/user-attachments/assets/6c13b264-74b1-4fd9-b622-65758de18922)

* Verify that the pod is running the `sleep infinity` process, as specified in the Dockerfile, by typing the following command in the terminal:

  ```
  ps aux | grep sleep
  ```

  ![image](https://github.com/user-attachments/assets/25cdf287-6141-4b66-abad-cc9435412a57)

### 5. Set up a GitHub webhook

GitHub webhooks allow external services to be notified when certain events happen. To automatically deploy a new pod when updates to the GitHub Dockerfile occur, you must configure your GitHub repo with the webhook that OpenShift provides as part of the `BuildConfig`. This is best achieved by using the OpenShift web console.

* From the menu, select __Builds__.

* On the Build Configs page, select the build configuration of your application.

  ![image](https://github.com/user-attachments/assets/10fb79a1-f0ff-41e3-bb1a-56c0474fe267)

* You are redirected to the detailed view of the build config. Go to the Webhooks section and click __Copy URL with Secret__ for the `Generic` webhook type.

  ![image](https://github.com/user-attachments/assets/9a109b62-bf43-4d9e-8680-1c92bbe87cba)

* Open your GitHub repo and navigate to __Settings__.

* Select __Webhooks__ from the Options menu and click __Add Webhook__.

  ![image](https://github.com/user-attachments/assets/cbf435a7-44c1-4973-b895-209b48487e46)

* On the __Add webhook__ page, paste the URL that you copied earlier into the __Payload URL__ field.

* In the __Content type__ field, select the `application/json` option.

* Click __Add webhook__.

  ![image](https://github.com/user-attachments/assets/a7a94be9-c3eb-46e2-abb2-7dbf12f5ccb1)

* Select your newly added webhook to see its details. The Recent Deliveries section displays an entry for a ping test that is prefixed with a tick mark, which indicates that the ping test was successful.

  ![image](https://github.com/user-attachments/assets/706465c1-4d05-4ec3-a109-9e61c7385944)

* Click the entry to view more details about the REST API call and the associated response for the ping test. A successful ping test indicates that GitHub is able to connect with your OpenShift cluster.

### 6. Make changes to your GitHub repository

In this step, you make a small change to the Dockerfile, change the `CMD` that is used to keep the container alive, and commit the change. This triggers a push event from GitHub to the OpenShift cluster, which causes OpenShift to rebuild the Docker image and redeploy the pod by using the newly built Docker image.

* Go to the Dockerfile in GitHub, edit the file by adding a comment to the first command and removing the comment from the second command, as shown in the following screen capture image.

  ![image](https://github.com/user-attachments/assets/41e54e18-c12c-4988-a8a3-f8863fefce53)

* Commit your changes.

* Go back to the OpenShift web console and open the Administrator view.

* Click __Builds__. Notice that a new build is automatically initiated, as shown in the following screen capture image. After this build is done, a new pod is created by using the newly built Docker image.

  ![image](https://github.com/user-attachments/assets/65e6db47-cd48-45a7-83d6-c8d72bc4058d)

* Select the __Pods__ view to see that the old pod is being terminated and new pod is being created.

  ![image](https://github.com/user-attachments/assets/bb90a17b-ca92-4ef7-8957-a97dc390d04c)

* Select the new pod and go to the terminal to verify that the new container is running the new process that is specified in the Dockerfile. Use the following command in the terminal:

  ```
  ps aux | grep sleep
  ```

* You receive output similar to the one shown in the following screen capture image.

  ![image](https://github.com/user-attachments/assets/4bb220de-9083-41a7-924e-63710b706642)

* If you go to your GitHub webhooks view, you can see a new entry within the Recent Deliveries section. The entry maps to the recent notification delivered by GitHub to OpenShift in response to the new commit in the repo.

  ![image](https://github.com/user-attachments/assets/42531ae5-f531-4356-b393-8f5a2a16b969)

## Summary

In this tutorial, you successfully demonstrated the OpenShift S2I capability by using a Dockerfile. You deployed a pod from a Dockerfile hosted in GitHub, set up the connection between OpenShift and GitHub by using a webhook, and tested that new code changes within the repository resulted in a new pod deployment within your OpenShift cluster.

---
draft: false
display_in_listing: true
title: "Create, scale, upgrade, and rollback an application on Red Hat OpenShift"
subtitle: "Manage an entire application lifecycle on Red Hat OpenShift"
meta_title: "Create, scale, upgrade, and rollback an application on Red Hat OpenShift"
authors:
  - email: "deepakcshetty@in.ibm.com"
    name: "Deepak C. Shetty"
  - email: "masa.abushamleh@ibm.com"
    name: "Masa Abushamleh"
editors:
  - name: Michelle Corbin
    email: corbinm@us.ibm.com
completed_date: "2021-02-22"
last_updated: "2024-02-28"
check_date: "2025-02-28"
time_to_read: 30
excerpt: "Explore how to execute some of the basic operations on Red Hat OpenShift: deploy, scale, update, and rollback your updates."
meta_description: "Explore how to execute some of the basic operations on Red Hat OpenShift: deploy, scale, update, and rollback your updates."
meta_keywords: "red hat openshift, ibm cloud, containers, docker"
primary_tag: containers
tags:
  - "cloud"
  - "red-hat"
components:
  - "docker"
  - "redhat-openshift-ibm-cloud"
  - "cloud-ibm"
related_content:
  - type: series
    slug: openshift-101
  - type: tutorials
    slug: debug-and-log-your-kubernetes-app
---

This tutorial shows you how to manage a basic application lifecycle on Red Hat® OpenShift®. You learn to create and deploy an application from an existing Docker image, scale it according to workload demands, update it to a newer version, and finally rollback the application to a previous version.

## Prerequisites

For this tutorial, you need:

* Red Hat OpenShift Cluster 4.3 or above on IBM Cloud
* [IBM Cloud Shell](https://shell.cloud.ibm.com/shell?cm_sp=ibmdev-_-developer-_-trial)

## Estimated Time

It will take you around 30 minutes to complete this tutorial.

## Steps

1. [Create an application from an existing Docker image](#create-an-application-from-an-existing-docker-image)
1. [Scale the application using replicas](#scale-the-application-using-replicas)
1. [Update the application](#update-the-application)
1. [Rollback the application](#rollback-the-application)

## Create an application from an existing Docker image

1. In IBM Cloud Shell, create a new project using the following command:

    ```
    oc new-project guestbook-project
    ```

1. Create a new deployment resource using the [ibmcom/guestbook:v1](https://hub.docker.com/r/ibmcom/guestbook/tags) Docker image in the project that you just created:

    ```
    oc create deployment myguestbook --image=ibmcom/guestbook:v1
    ```

1. This deployment creates the corresponding pod that's in running state. Use the following command to see the list of pods in your namespace:

    ```
    oc get pods
    ```

    ![image](https://github.com/user-attachments/assets/ced69b5b-0bf0-4c04-b9bf-4ad382889a81)


1. Expose the deployment on port 3000:

    ```
    oc expose deployment myguestbook --type="NodePort" --port=3000
    ```

1. To view the service that you just exposed, use the following command:

    ```
    oc get service
    ```

  ![image](https://github.com/user-attachments/assets/207d32d7-bb7f-4108-881d-e7bd293794ec)


  Note that you can access the service inside the pod using the `<Node IP>:<NodePort>`, but in the case of OpenShift on IBM Cloud, the NodeIP is not publicly accessible. You can use the built-in Kubernetes terminal available in IBM Cloud, which spawns a Kubernetes shell that is part of the OpenShift cluster network; you can access `NodeID:NodePort` from that shell.

1. To make the exposed service publicly accessible, you need to create a public router. First, go to **Networking > Routes** from the Administrator Perspective on the web console, and then click **Create Route**.

1. Fill in the information as follows and click **Create** (you can leave all but the following fields empty):

  * **Name**: myguestbook
  * **Service**: myguestbook
  * **Target Port**: 3000&#8594;3000(TCP)

    ![image](https://github.com/user-attachments/assets/fc47f50b-185f-4e2a-afb5-b9a2f8c7fd0b)

    Once you create the route, you are redirected to the Route Overview, where you can access the external route from the URL under Location as shown in the following screenshot. You have created the Route successfully, and you have a publicly accessible endpoint URL that you can use to access your guestbook application.

    ![image](https://github.com/user-attachments/assets/c1783fd8-814b-47f5-bd06-45612b81424d)

    If you click on the URL, you are redirected to a page that looks like the following:

    ![image](https://github.com/user-attachments/assets/9465cbd3-c10b-46b1-a197-f56b827933fb)

Now that you have successfully deployed the application using the existing Docker image, you are ready to scale and rollback your application.

## Scale the application using replicas

In this section, you scale the application by creating replicas of the pod you created in the first section. By having multiple replicas of a pod, you can ensure that your deployment has the available resources to handle an increasing load on your application.

1. In IBM Cloud Shell, increase the capacity from a single running instance of guestbook up to 5 instances:

    ```
    oc scale --replicas=5 deployment/myguestbook
    ```

2. Check the status of the deployment:

    ```
    oc rollout status deployment myguestbook
    ```

    ![image](https://github.com/user-attachments/assets/d1dc932f-eda8-4600-b5a3-e678646cca85)

3. Verify that you have 5 pods running after running the `oc scale` command:

    ```
    oc get pods -n guestbook-project
    ```

    ![image](https://github.com/user-attachments/assets/78974bfe-75dc-4296-a01a-5920d19727b7)

    ![image](https://github.com/user-attachments/assets/2a357fd4-1612-49e1-bcc3-64f92931d10d)

## Update the application

With OpenShift, you can perform a rolling upgrade of your application to a new container image. With a rolling upgrade, you can update the running image and undo a rollout if you discover a problem during or after the deployment. In this section, you upgrade the image with a v1 tag to a new version with a v2 tag.

1. In IBM Cloud Shell, update your deployment using the v2 image:

    ```
    oc set image deployment/myguestbook guestbook=ibmcom/guestbook:v2
    ```

1. Check the status of the rollout:

    ```
    oc rollout status deployment/myguestbook
    ```

    ![image](https://github.com/user-attachments/assets/bdabe615-8736-4deb-91a8-d2f7d240c058)

1.  Get a list of pods (newly launched) as part of moving to the v2 image:

    ```
    oc get pods -n guestbook-project
    ```

    ![image](https://github.com/user-attachments/assets/7bb5f83c-25af-4c5f-9a93-d13ad536bcd3)

1.  Copy the name of one of the new pods and use `oc describe pod` to confirm that the pod is using the v2 image like the following screenshot:

    ```
    oc describe pod <pod-name>
    ```

    ![image](https://github.com/user-attachments/assets/6d48eb79-35ce-4c87-845f-762d5d498e25)

    Notice that the service and router objects remain unchanged when you change the underlying Docker image version of your pod.

1. Perform a hard refresh on the public router URL to see that the Pod is now using the v2 image of the guestbook application. (In most browsers, pressing Ctrl+F5 performs a hard refresh.)

    ![image](https://github.com/user-attachments/assets/50c4680b-67e8-47b0-ad56-228e875cdf73)

## Rollback the application

When performing a rollout, you can see references to old replicas and new replicas. In this project, the old replicas are the original 5 pods you deployed in the second section when you scaled the application. The new replicas come from the newly created pods with the new image. All these pods are owned by the deployment. The deployment manages these two sets of pods with a resource called ReplicaSet.

1. To see the guestbook ReplicaSets, use the following command:

    ```
    oc  get replicasets -l app=myguestbook
    ```

    ![image](https://github.com/user-attachments/assets/05a0f39b-cdf6-4a08-9c08-4413a584ed14)

1. Undo your latest rollout using the following command:

    ```
    oc rollout undo deployment/myguestbook
    ```

    ![image](https://github.com/user-attachments/assets/a5136062-efc2-4b79-ad3f-df2ff824e248)

1. Get the status of Undo deployment to see the newly created pods as part of undoing the rollout:

    ```
    oc rollout status deployment/myguestbook
    ```

    ![image](https://github.com/user-attachments/assets/f44cb2b6-334a-4682-87cb-06a98fd1184b)

1. Get the list of the newly created pods as part of undoing the rollout:

    ```
    oc get pods
    ```

   ![image](https://github.com/user-attachments/assets/ed9d22dc-f225-4f20-8553-c99111178b99)

1. Copy the name of one of the pods and use it in the `oc describe pod` command to view the image and its version used in the pod:

    ```
    oc describe pod <pod-name>
    ```

    ![image](https://github.com/user-attachments/assets/565cbaea-8685-443a-bfa3-5d411e2b4466)

1. After undoing the rollout, check the ReplicaSets, and you will notice that the old replica set is now active and manages the 5 pods using the following command:

    ```
    oc get replicasets -l app=myguestbook
    ```

    ![image](https://github.com/user-attachments/assets/5e8aa22c-d1ae-4ca9-a23f-022395231d89)

1. To view changes, make sure to hard refresh your browser pointing at the router URL (as before) to see if v1 version of the application is running:

    ![image](https://github.com/user-attachments/assets/d77c25ca-9a64-4e93-b085-9f23e5cd30c7)

## Summary

Now that you know how to deploy, scale, update, and rollback your application on OpenShift, you can use this knowledge to deploy applications from any existing Docker images, scale them to handle an increasing workload, update them as newer versions of Docker images are released, and rollback to older versions in case of any bugs/issues in the newer version.

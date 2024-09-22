---
# Related publishing issue: https://github.ibm.com/IBMCode/IBMCodeContent/issues/5287
# https://w3.ibm.com/developer/contentcreatortool/cct/developer/master/Tutorials/

draft: false

title: "Enforce resource consumption limits using OpenShift resource quotas"
subtitle: "Configure resource limits for your applications and projects on Red Hat OpenShift"
meta_title: "Enforce resource consumption limits using OpenShift resource quotas"
authors:
  - name: Deepak C Shetty
    email: "deepakcshetty@in.ibm.com"
  - name: Masa Abushamleh
    email: "masa.abushamleh@ibm.com"

completed_date: "2021-06-03"
last_updated: "2021-06-03"
# archived due to low page views
archive_date: "2023-12-15"
check_date: "2024-09-15"

time_to_read: 30
excerpt: "Configure resource limits for your applications and projects on Red Hat OpenShift."
meta_description: "Configure resource limits for your applications and projects on Red Hat OpenShift."
meta_keywords: "containers, openshift, resources, quotas"

primary_tag: "containers"

components:
  - "redhat-openshift-ibm-cloud"
related_content:
  - type: series
    slug: openshift-101
  - type: tutorials
    slug: multitenancy-and-role-based-access-control

related_links:
  - title: "Resource quotas per project"
    url: "https://docs.openshift.com/container-platform/4.7/applications/quotas/quotas-setting-per-project.html"
---
## Introduction

When several users or teams share a cluster with a fixed number of nodes, it is possible one user or team could use more than its fair share of resources. You can avoid this situation by setting a resource quota, which limits the aggregate resource consumption per project or pod/container. A resource quota limits the quantity of objects that can be created in a project by type, as well as the total amount of compute and storage resources that resources can consume in that project.

In Red Hat OpenShift, you can manage constraints by using the `ResourceQuota` object on a project level or by using the `LimitRange` object on a pod/container level. With limit ranges, you can specify the minimum and maximum usage of memory and CPU. With resource quotas, you can specify the amount of memory and CPU and number of pods, replication controllers, services, secrets, and persistent volume claims. A resource quota is enforced on a project or pod/container when there is `ResourceQuota`/`LimitRange` specified. This tutorial shows you how to set different resource limits for your applications and projects on Red Hat OpenShift and explores what happens when you try to exceed the quota.

## Prerequisites

For this tutorial, you need:

* [IBM Cloud account](https://cloud.ibm.com/login?cm_sp=ibmdev-_-developer-_-trial)
* A Red Hat OpenShift cluster 4.3 or higher on IBM Cloud
* [IBM Cloud Shell](http://shell.cloud.ibm.com?cm_sp=ibmdev-_-developer-_-trial)

## Estimated time

It will take you around 30 minutes to complete this tutorial.

## Log in to the CLI and create a project

1. From the OpenShift web console on IBM Cloud, click your username at the top right, and then click **Copy Login Command**.
1. Click **Display Token**, and copy the `oc login` command.
1. In your terminal, enter `ibmcloud`, and then paste your `oc login` command.
1. Enter the following command to create a project named `quota-demo`:

    ```
    oc new-project quota-demo
    ```

## Configure a pod quota for a namespace

1. Enter the following command to create a `ResourceQuota` for the project that limits the number of pods to 2:

    ```
    oc apply -f https://raw.githubusercontent.com/nerdingitout/oc-quota/main/project-quota.yaml
    ```

    You see the message `resourcequota/pod-demo created`.

1. Enter the following command to view details about `ResourceQuota`:

    ```
    oc get resourcequota
    ```

    You see that `ResourceQuota` was created with 2 pods in the project:

    

1. You can also view the resource quota on the web console from the Administrator Perspective. From the web console navigation menu, click <b>Administration &#8594; Resource Quotas</b>. You see the Resource Quotas page, with the pod-demo listed, as shown in the following image:

    ![resourcequota webconsole](images/105811063-791be780-5fc5-11eb-8aa5-7950c7dc3b4a.JPG)

1. Click **pod-demo** to view the details of the resource quota. If you scroll down to the Resource Quota Details section for the pods resource type, you can view the used and maximum resources. Because you haven't deployed any pods yet, it shows that 0 pods have been used.

    ![quota details](images/105811625-5b02b700-5fc6-11eb-8a95-06414df4b7d1.JPG)

## Verify that the pod quota works

1. From the CLI, enter the following to deploy a simple application:

    ```
    oc create deployment myguestbook --image=ibmcom/guestbook:v1 -n quota-demo
    ```

    You see the message `deployment.apps/myguestbook created`.

1. Enter `oc get pods`, and the status shows that the pod is in running state:

    ![image](images/106388021-06ac6c80-63f6-11eb-9565-558f020633e2.png)

1. On the Resource Quota Details section on the web console, you can see that the number of used pods is now 1, as expected. This shows that the `ResourceQuota` has correctly counted the pod against the specified quota.

    ![image](images/106388036-14fa8880-63f6-11eb-8c56-2411c358184d.png)

1. You can scale the application from both the web console and the command line. In this step, you use the web console to scale the application to 2 replicas. From the navigation menu, click **Workloads &#8594; Deployments**, and then click the `myguestbook` deployment:

    ![deployments](images/106388296-458ef200-63f7-11eb-8d7b-e0b354d7c2f0.JPG)

1. On the Deployment Details page, you can click the upper arrow to increase the number of replicas and the down arrow to decrease the number of replicas. Clck the upper arrow to increase the number of replicas to 2:

    ![scale pods](images/106388343-7bcc7180-63f7-11eb-9372-2c11b8ac2081.JPG)

1. From the navigation menu, click **Administration &#8594; Resource Quotas**, and then click **pod-demo**. You can see that the number of used pods has increased to 2:

    ![image](images/106388411-d534a080-63f7-11eb-813f-515a2266fc20.png)

1. From the command line, let's try exceeding the quota by adding another pod:

    ```
    oc create deployment myguestbookv2 --image=ibmcom/guestbook:v2 -n quota-demo
    ```

    You see the message `deployment.apps/myguestbookv2 created`.

1. Enter `oc get pods`, and you see that there are only 2 pods running, even though you created a total of 3 pods:

    ![image](images/106388687-2beeaa00-63f9-11eb-8518-0b487b8efde6.png)

1. Go back to the web console and click **Workloads &#8594; Deployments**. You see the status of the myguestbook2 deployment shows 0 of 1 pods:

    ![deployments 2](images/106388766-6ce6be80-63f9-11eb-9fc8-2ce9cb4a15d2.JPG)

1. Click the **myguestbook2** deployment to view more details. You can see that the pod hasn't been deployed.

    ![image](images/106388869-db2b8100-63f9-11eb-82a4-b1a54011c9d9.png)

1. Scroll to the Conditions section, which shows that the pod has failed with a message that the quota has been exceeded.

    ![failed pod](images/106388947-2ba2de80-63fa-11eb-9b77-1c6db42271eb.JPG)

1. To prepare for the next step, let's delete all of the resources and the resource quota you created. From the command line, enter the following commands to delete all resources for the 'myguestbook' and 'myguestbookv2' deployments:

    ```
    oc delete all --selector app=myguestbook
    oc delete all --selector app=myguestbookv2
    ```

    You see messages that the pods and deployments have been deleted:

    ![delete all](images/106389819-d9b08780-63fe-11eb-87a0-90d5df2ecbbb.JPG)

## Configure memory and CPU for a namespace

1. Create a new `ResourceQuota` for memory and CPU using the following `oc apply` command:

    ```
    oc apply -f https://raw.githubusercontent.com/nerdingitout/oc-quota/main/quota-mem-cpu.yaml
    ```

    You see the message `resourcequota/mem-cpu-demo created`.

    The YAML file contains the following details for the quota:
    ```
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: mem-cpu-demo
    spec:
      hard:
        requests.cpu: "1"
        requests.memory: 1Gi
        limits.cpu: "2"
        limits.memory: 2Gi

    ```

1. When defining the quota for memory and CPU, you are defining minimum (requests) and maximum (limits) resources to be used. Once created, you can view the resource quota on the web console. Click **Administration &#8594; Resource Quotas**, and then click **mem-cpu-demo**. You can see that the resources haven't been used yet:

    ![image](images/106389845-095f8f80-63ff-11eb-93d6-325469edbfca.png)

This `ResourceQuota` places the following requirements on the `quota-demo` namespace:

* Every container must have a memory request, memory limit, CPU request, and CPU limit.
* The memory request total for all containers must not exceed 1 GiB.
* The memory limit total for all containers must not exceed 2 GiB.
* The CPU request total for all containers must not exceed 1 core.
* The CPU limit total for all containers must not exceed 2 cores.

## Verify that the CPU and memory quotas work

1. Create an Nginx application using the following command:

    ```
    oc apply -f https://raw.githubusercontent.com/nerdingitout/oc-quota/main/nginx-app.yaml
    ```

    You see the message `pod/nginx-app created`.

    The yaml definition specifies the memory and CPU for the pod, as shown here:

    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx-app
    spec:
      containers:
      - name: quota-mem-cpu-demo-ctr
        image: nginx
        resources:
          limits:
            memory: "800Mi"
            cpu: "800m"
          requests:
            memory: "600Mi"
            cpu: "400m"
    ```

1. Because the requests and limits for the CPU and memory are within the quota, the pod should be created successfully. Enter `oc get pods` to see that nginx-app is running:

    ![image](images/106390264-ee8e1a80-6400-11eb-8311-a4a7dacf5e3c.png)

1. On the web console, look at the Resource Quota Details section to see that the Nginx pod’s CPU and memory requests and limits are correctly accounted for against the `ResourceQuota`:

    ![image](images/106390279-082f6200-6401-11eb-83cc-33496b634321.png)

Now, let's try to exceed the quota by creating a new application using the `redis docker` image. The yaml definition includes the following details. Keep in mind that it will exceed the quota, therefore the application won't be created.

```
apiVersion: v1
kind: Pod
metadata:
  name: redis-app
spec:
  containers:
  - name: quota-mem-cpu-demo-2-ctr
    image: redis
    resources:
      limits:
        memory: "1Gi"
        cpu: "800m"
      requests:
        memory: "700Mi"
        cpu: "400m"
```

1. Use the following command to create the application:

    ```
    oc apply -f https://raw.githubusercontent.com/nerdingitout/oc-quota/main/redis-app.yaml
    ```

    You receive the following error, because the application exceeded the quota:

    ![image](images/106390512-3b262580-6402-11eb-90bc-2617cda77f37.png)

1. You can increase the resource quota to fit in the new application. From the web console, click **YAML** on the mem-cpu-demo Resource Quota Details page. Copy and paste the following lines of code between lines 65 and  71 and then click **Save**:

    ```
    spec:
      hard:
        limits.cpu: '4'
        limits.memory: 4Gi
        requests.cpu: '1'
        requests.memory: 2Gi
    ```
    The following image shows the updated lines of code in the web console:

    ![image](images/106390761-5fcecd00-6403-11eb-8521-105abff0aeb2.png)

1. Enter the following in the CLI to attempt to create the same redis application again.

    ```
    oc apply -f https://raw.githubusercontent.com/nerdingitout/oc-quota/main/redis-app.yaml
    ```

    This time, you receive the message `pod/redis-app created`.

1. Enter `oc get pods`, and you see that the new application is running successfully.

    ![image](images/106390869-ebe0f480-6403-11eb-9621-82f169bea672.png)

1. Go back to the mem-cpu-demo Details tab, and you see that it has changed and now the consumed resources are counted for both applications:

    ![image](images/106390818-b76d3880-6403-11eb-9bf8-db3d9d7af3bd.png)

## Summary

In this tutorial, you learned how to create resource quotas for the number of pods, CPU, and memory usage limits. You can apply quotas to even more resources in OpenShift. Explore the <a href="https://docs.openshift.com/container-platform/4.7/applications/quotas/quotas-setting-per-project.html" target="_blank" rel="noopener noreferrer nofollow">OpenShift resource quota</a> documentation to learn how to apply quotas to object counts and storage and compute resources using what you learned from this tutorial.

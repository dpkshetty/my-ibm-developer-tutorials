---
# Related publishing issue: https://github.ibm.com/IBMCode/Code-Tutorials/issues/5216
# https://w3.ibm.com/developer/contentcreatortool/cct/developer/master/Tutorials/

draft: false
display_in_listing: true
title: "Multitenancy and role-based access control in Red Hat OpenShift"
subtitle: "Isolate your workloads with role binding"
meta_title: "Red Hat OpenShift multitenancy and role-based access control"
authors:
  - email: "deepakcshetty@in.ibm.com"
    name: "Deepak C Shetty"
  - email: "Masa.Abushamleh@ibm.com"
    name: "Masa Abushamleh"
editors:
  - email: binumesh@in.ibm.com
    name: Bindu Umesh
completed_date: "2020-12-17"
last_updated: "2020-12-17"
check_date: "2024-12-01"
time_to_read: 30
excerpt: "Achieve multitenancy in your cluster using Red Hat OpenShift Container
  Platform role-based access control and role bindings."
meta_description: "Achieve multitenancy in your cluster using Red Hat OpenShift Container
  Platform role-based access control and role bindings."
meta_keywords: "red hat, openshift, role binding, multitenant, ibm cloud, multitenancy, rbac"
primary_tag: "containers"
tags:
  - "cloud"
components:
  - "redhat-openshift-ibm-cloud"
social_media_meta: "Achieve multitenancy in your cluster using Red Hat OpenShift Container
  Platform role-based access control and role bindings."
related_content:
  - type: learningpaths
    slug: multitenancy-red-hat-openshift

also_found_in:
  - "learningpaths/multitenancy-red-hat-openshift/"

---
## Introduction

Multitenancy is an architecture in which a single instance of a software application serves multiple customers (tenants), where each tenant's data is logically separated from the other tenants. Role-based access control (RBAC) is a method of restricting access based on the roles of the users in the cluster, where users have the rights to access the resources they need. In RBAC, role bindings define which users are entitled to view and manage authorized resources, and they grant the permissions defined in a role to a user or a group. While a role binding grants permissions at a project scope, cluster role binding grants permissions at a cluster level.

This tutorial shows how you can achieve multitenancy by using Red Hat OpenShift Container Platform RBAC and role binding. It walks you through creating a project in OpenShift, adding users to the cluster, and then using role bindings to limit the users to specific projects (namespaces) to ensure that their workloads are isolated from other users. Their resources and views are separated from other users on the same OpenShift cluster using the same set of shared hardware resources. You also learn how to impersonate a user, which administrators can use to troubleshoot a user's issues on the cluster. In this tutorial, we use impersonation to show you the cluster view of a user.

## Prerequisities

For this tutorial, you need:

* A Red Hat OpenShift 4.3 cluster or above on IBM Cloud
* [IBM Cloud Shell](https://shell.cloud.ibm.com/shell?cm_sp=ibmdev-_-developer-_-trial)

## Estimated Time

It will take you around 30 minutes to complete this tutorial.

## Steps

1. [Log in and create projects](#log-in-and-create-projects)
1. [Create users](#create-users)
1. [Create role bindings](#create-role-bindings)
1. [Impersonate user and deploy application](#impersonate-user-and-deploy-application)
1. [Create and deploy the pod](#create-and-deploy-the-pod)
1. [Switch to another user](#switch-to-another-user)

## Log in and create projects

We create users and projects with the IBM Cloud Shell because you cannot create local users from the web console.

1. Log in to the Red Hat OpenShift for IBM Cloud web console for your cluster. Click your username in the header and then click **Copy Login Command**. Click **Display Token** and copy the `oc login` command and paste it in your terminal.

  ![image](https://github.com/user-attachments/assets/ffaf91d9-87ab-4516-9ea4-48085ed02e88)

1. Create two projects, one for each user in our example, by copying the following commands in your terminal:

  ```
  oc create namespace my-first-project
  ```

  You see a message that `my-first-project` has been created.

  ```
  oc create namespace my-second-project
  ```

  You see a message that `my-second-project` has been created.

## Create users

1. Copy and paste the following commands on the command line to create two users:

  ```
  oc create user first-user --full-name="first user"
  ```

  You see a message that `first-user` has been created.

  ```
  oc create user second-user --full-name="second user"
  ```

  You see a message that `second-user` has been created.

  As Administrator, you can see all the users, projects, and pods associated with the cluster.

1. In the web console, click **User Management** > **Users** to see the new users listed.

  ![image](https://github.com/user-attachments/assets/7806daa2-693f-4950-b598-a1b3646c719f)

1. Click **Home > Projects** to view all the projects in the cluster.

  ![image](https://github.com/user-attachments/assets/2c07eb95-cc18-4e5e-81a3-200dfb13a06b)

1. Click **Workloads > Pods** to view all pods (make sure that you click **Project** > **all projects** to list all of the projects).

  ![image](https://github.com/user-attachments/assets/619499c9-a9f5-44d5-8154-b5535dbdb87a)

Version 4.3 of the OpenShift web console UI added the ability to [spoof other users and groups](https://www.openshift.com/blog/openshift-4-3-spoofing-a-user). Let's impersonate one of the users to see what that user has access to at this point.

Click **User Management > Users** and then click the vertical ellipsis button for **first-user**. Click **Impersonate User first-user**.

![image](https://github.com/user-attachments/assets/6eb7e840-0a5f-4b9c-b779-a3f37006b5f4)

If you click **Home > Projects**, you can see that `first-user` can't view any projects/pods or create any new projects/pods. That's because `first-user` doesn't have a role binding yet, which means `first-user` does not have any privileges and/or permissions to create, view, or manage projects and pods.

![image](https://github.com/user-attachments/assets/bc343a4b-121f-4da3-97d2-2545ccba3ad6)

![image](https://github.com/user-attachments/assets/a3ffba70-2d5e-44fd-bb6e-01d4529b8cab)

Click **Stop Impersonation** in the Impersonating User header to stop impersonating `first-user` and switch back to Administrator.

![image](https://github.com/user-attachments/assets/ecbc7014-0c2d-42ba-bf6e-be58d8f8411a)

## Create role bindings

In this step, you create role bindings to give users access to their respective projects. You can easily create role bindings through the web console.

1. Click **User Management > Role Bindings**.
1. Click **Create Binding**.
1. On the **Create Role Binding** page, fill out the form as follows and click **Create**:

  * **Binding Type**: Namespace Role Binding (RoleBinding)
  * **Name**: first-user-rb
  * **Namespace**: my-first-project
  * **Role Name**: cluster-admin
  * **Subject**: User
  * **Subject Name**: first-user

  ![image](https://github.com/user-attachments/assets/4772b397-a199-41e9-a5c7-9a2323097980)

1. Click **User Management > Users** and click **first-user**. Click the **Role Bindings** tab, and you can see that the role `cluster-admin` has been successfully associated with `first-user` through the `first-user-rb` role binding for the namespace `my-first-project`.

![image](https://github.com/user-attachments/assets/f4447463-15d9-4d09-a95f-3924fb34a95a)

## Impersonate user and deploy application

In this step, you impersonate `first-user` like you did previously, but this time, `first-user` has admin permissions to `my-first-project`, which means `first-user` can do everything an admin can do on that namespace.

1. Click **User Management > Users** and then click **first-user**. Click **Actions > Impersonate Users first-user**.

  Notice that now `first-user` can view the `my-first-project` namespace. It is the only project this user can view because `first-user` was limited to the `my-first-project` namespace.

  ![image](https://github.com/user-attachments/assets/b995974a-bbd1-48b9-8b6f-27687a9a2caa)

1. Click **Workloads > Pods** to see that `first-user` has access to the pods as well (and can create pods), but no pods have been created yet.

  ![image](https://github.com/user-attachments/assets/2c71e3bf-65f6-43cd-a991-762937b36b20)

## Create and deploy the pod

In this step, you create a pod as `first-user` in the `my-first-project` namespace.

1. Switch to the Developer Perspective on the web console.
1. Click **Topology**, and then click **Container Image**.

  ![image](https://github.com/user-attachments/assets/ccbde421-79bf-436d-bf66-66bfc473c38f)

1. On the **Deploy Image** page, enter `ibmcom/guestbook:v1` in the **Image name from external registry** field. (You should see a **Validated** messsage. Keep the default settings and click **Create**.

  ![image](https://github.com/user-attachments/assets/0f55f26f-8462-459c-826c-051b493fbe3f)

Once you create the pod, you are redirected to the **Topology** view, which shows the pod. The light blue circle around the pod turns dark blue once it successfully builds. As `first-user`, you can successfully deploy a containerized application (guestbook:v1) in the project `my-first-project` and can view the pods and deployment resources associated with the application.

![image](https://github.com/user-attachments/assets/7811c3aa-3bdb-427f-82e3-8a1c016efc74)

You can access the deployed application by clicking on the small square on the top corner of the pod, which opens Guestbook-v1 in a new window.

![image](https://github.com/user-attachments/assets/bc329133-6526-44ad-a905-8ee7e24a4c94)

In the web console, switch back to the Administrator perspective. Notice that the new pod has been added to the list of pods for the project `my-first-project`.

![image](https://github.com/user-attachments/assets/4e81f01a-8b38-44e5-8ff1-f365cea6ad68)

## Switch to another user

Now, let's stop the impersonation for `first-user`, and switch to `second-user` to verify if `second-user` can see the pods `first-user` created.

1. Click **Stop Impersonation** in the Impersonating User header to stop impersonating `first-user` and switch back to Administrator.
1. Click **User Management > Users** and then click the vertical ellipsis button for **second-user**. Click **Impersonate Users second-user**.

As expected, `second-user` cannot see the project `my-first-project` because `second-user` is not authorized.

![image](https://github.com/user-attachments/assets/b8b05556-e295-4978-a811-a681951d836a)

## Summary

Thanks to role bindings in OpenShift, we have isolated the workloads across users on the cluster and achieved multitenancy. As a real-world use case, a managed service provider could use this feature to provide logical isolation to its customers while using the same OpenShift Container Platform cluster and the same set of shared hardware resources.

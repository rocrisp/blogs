
Welcome to this deep dive into l5-operator! 

In this blog post, we will explore the inner workings of the l5-operator, a Kubernetes operator that manages the lifecycle of Bestie resources. 

We'll delve into its architecture, key components, and the core logic behind its operations. 

Whether you're a beginner looking to understand Kubernetes operators or an experienced developer seeking insights into l5-operator's implementation details, this deep dive is for you.

Table of Contents:

[What is a Kubernetes Operator?](#What-is-a-Kubernetes-Operator?)

[Introducing l5-operator](#Introducing-l5-operator)

[Architecture Overview](#Architecture-Overview)

[Reconciliation: The Heart of l5-operator](#internal/sub_reconcilers/)

[Controllers](#controllers) and [Custom Resource Definitions (CRDs)](#Custom-Resource-Definition)

[Managing Deployments with DeploymentReconciler](#Managing-deployments)

[Database Seeding and Condition Updates](#Database-seeding)

[Deploying l5-operator](#Deploying-l5-operator)

[Best Practices and Lessons Learned](#Best-Practices-and-Lessons-Learned)

[Conclusion](#Conclusion)

#### What is a Kubernetes Operator?
A Kubernetes Operator is a way to automate and standardize the deployment and lifecycle management of complex applications on Kubernetes. 

The Operator **Pattern** involves creating a **Custom Resource Definition** to define a new custom resource type, implementing a **Custom Controller** to manage instances of that resource, and encoding **application domain knowledge** into the controller.

This **Pattern** leverages the extensibility of Kubernetes, allowing developers to manage applications and services more effectively and with less operational overhead. This can greatly reduce the manual work involved in deploying, scaling, updating, and maintaining those applications and services.

#### Introducing l5-operator
The L5 Operator is a Kubernetes Operator that's responsible for deploying and managing the lifecycle of the [BESTIE](https://github.com/makon57/bestie) application. 

To initiate Bestie application(known as the Operand), you need to first populate the database with the command: ```["/bin/sh",  "-c", "flask db migrate & flask db upgrade & flask seed all"]```Next, initiate the Postgress database. Once that's active, you can launch the application.

The L5-operator streamline the process by taking over all manual tasks, enabling the launch of the Operand with ease and efficiency.

More on the L5-operator and it's capabilities can be found [here](https://github.com/opdev/l5-operator-demo/tree/main#readme)

#### Architecture Overview
In this section, we'll dive into the architectural components of l5-operator. We'll explore its high-level structure, including controllers, custom resource definitions (CRDs), and the interactions between different components.

**api/v1:**
This directory holds the definitions for the Custom Resource (CR) that your operator will be watching.

**bestie_types.go:**

  It defines a new custom resource named Bestie. The Bestie resource has a specification and a status, which represent the desired and observed state of the resource.
  
- Here's a summary of the key components:

  - **BestieSpec Struct** defines the spec for the Bestie custom resource. It includes fields such as Size, Image, Version, and MaxReplicas for the desired state of Bestie. 
  - **BestieStatus Struct** defines the status for the Bestie custom resource. It includes PodStatus, AppVersion, and Conditions for the observed state of Bestie.


##### **controllers/:**
This directory tipically contains the controllers that react when Bestie CR is created, updated, or deleted.

**bestie_controller.go**:
  
  Defines the controller for the Bestie CR as part of a Kubernetes Operator. This controller watches for changes to Bestie objects and reconciles the state of the system to match the desired state as expressed in the Bestie objects. It uses a set of sub-reconcilers to handle different aspects of the Bestie's desired state.
  
- Here's a summary of the key components:
   - **BestieReconciler Struct:** This struct is the main controller for the Bestie CR. It includes a client for interacting with the Kubernetes API, a runtime.Scheme for converting between different API versions, and an EventRecorder for recording Kubernetes events.
   - **Reconcile Method:** This is the core method of the controller. It's called whenever a Bestie object is created, updated, or deleted. The method fetches the latest state of the Bestie object, and then runs a list of sub-reconcilers to reconcile the state of the system with the desired state.
   - **Sub-Reconcilers:** The Reconcile method runs a list of sub-reconcilers, each responsible for a specific aspect of the Bestie object. For example, one sub-reconciler might handle creating a Deployment for the Bestie, while another might handle setting up a Service.
   - **SetupWithManager Function:** This function sets up the BestieReconciler with the controller manager. It specifies that this controller manages Bestie objects and owns Deployments, Services, Ingresses, and HorizontalPodAutoscalers related to the Bestie.



**config/:**

This directory contains configuration files for deploying your operator and its CRDs to a cluster.

#### **Custom Resource Definition**
**config/crd/bases/pets.bestie.com_besties.yaml** is a Kubernetes Custom Resource Definition (CRD) file for the Bestie resource, which is part of a Kubernetes Operator.

- Here's a summary of the key components:
    - **Kind:** The kind of the Kubernetes resource is CustomResourceDefinition.
    - **Metadata:** The metadata section provides information about the CRD, including its name (besties.pets.bestie.com).
    - **Spec:** The spec section defines the specification for the Bestie CR. It includes the following fields:
    - **group:** The group under which the resource is defined (pets.bestie.com).
    - **names:** The names to use for the resource in the API (Bestie, BestieList, besties, bestie).
    - **scope:** Specifies that the Bestie resource is namespaced, which means it exists within a specific namespace.
    - **versions:** Defines the available versions for the resource, the validation schema for the resource, and other version-specific information.
    - **OpenAPIV3Schema:** This section provides a schema for the Bestie resource, defining what fields it can have and their types. This includes fields for spec (the desired state) and status (the observed state).
    - **Status:** The status section is used by Kubernetes to store information about the state of the CRD itself, like the names it has been accepted under and the versions that are stored in etcd.

    In summary, this file defines a new custom resource named Bestie for use in a Kubernetes Operator. This CRD provides the structure for Bestie objects, which users can create, update, and delete. The Operator's controller will then reconcile the state of the system to match the desired state as expressed in the Bestie objects.
    
**config/rbac**

This directory contains configuration files related to Role-Based Access Control (RBAC). RBAC is a method of regulating access to a Kubernetes cluster's resources based on the roles of individual users within your organization.

Here is what you might find in the config/rbac directory:

- Role YAML Files: These files define the permissions that the Operator needs to function properly. They specify what actions (verbs) the Operator can perform on which resources.
- RoleBinding or ClusterRoleBinding YAML Files: These files bind the above roles to specific subjects (users, groups, or service accounts). The Operator's pod usually runs under a specific service account, and this service account is granted the necessary permissions through a RoleBinding or ClusterRoleBinding.
- ServiceAccount YAML File: This file creates a ServiceAccount that the Operator's pod will run under.

The purpose of separating these files into a config/rbac directory is to provide a clear, dedicated location for all access control configuration. This makes the project easier to understand and manage.

When deploying the Operator with tools like kustomize or kubectl, these RBAC configuration files will be applied to give the Operator the necessary permissions to interact with the Kubernetes API server. Without these permissions, the Operator would not be able to monitor and manipulate the Kubernetes resources that it manages.

**config/rbac/role.yaml** is a Kubernetes configuration file that defines a ClusterRole for use in a Kubernetes cluster. A ClusterRole is a type of Kubernetes Role-Based Access Control (RBAC) object that defines permissions at the cluster level, rather than within a specific namespace.

- Here's a summary of the key components:
    - **Kind:** The kind of the Kubernetes resource is ClusterRole.
    - **Metadata:** The metadata section provides information about the ClusterRole, including its name (manager-role).
    - **Rules:** The rules section defines the permissions granted by this ClusterRole. Each rule specifies:
    - **apiGroups:** The group of the API that the rule applies to.
    - **resources:** The resources within the API group that the rule applies to.
    - **verbs:** The operations that are allowed on the resources.
        For example, one of the rules grants all ('*') verbs on the resource pods in the "" (core) API group. This means the ClusterRole has permission to create, read, update, and delete pods cluster-wide.

    This ClusterRole is intended for a controller of a Kubernetes Operator that manages Bestie resources.

    In summary, the role.yaml file defines a ClusterRole that provides a set of permissions at the cluster level. It corresponds to the controller, granting it the necessary permissions to watch and manipulate the resources it manages, including the Bestie custom resources and other related Kubernetes resources.

**role_binding.yaml** is a Kubernetes configuration file that defines a ClusterRoleBinding. This is a type of Role-Based Access Control (RBAC) object that grants permissions defined in a ClusterRole to a subject (user, group, or service account) at the cluster level.

- Here's a summary of the key components:
    - **Kind:** The kind of the Kubernetes resource is ClusterRoleBinding.
    - **Metadata:** The metadata section provides information about the ClusterRoleBinding, including its name (manager-rolebinding).
    - **RoleRef:** The roleRef section references the ClusterRole that the ClusterRoleBinding is associated with. In this case, it's associated with a ClusterRole named manager-role.
    - **Subjects:** The subjects section defines the subjects that the ClusterRoleBinding applies to. In this case, it applies to a ServiceAccount named controller-manager in the system namespace.

    In summary, the role_binding.yaml file defines a ClusterRoleBinding that grants the permissions defined in the manager-role ClusterRole to the controller-manager service account at the cluster level. This service account is likely used to run the Operator's controller, and the ClusterRoleBinding ensures that the controller has the necessary permissions to interact with the Kubernetes API server.

#### Deploying l5-operator
**bundle/:**
The bundle directory contains 3 subdirectories: manifests, metadata, and tests/scorecard.

- **Manifests:** This directory contains all the manifest files that OLM needs to deploy and manage the Operator. This usually includes a ClusterServiceVersion (CSV) file and one or more Custom Resource Definition (CRD) files. The CSV file contains the deployment specifications for the Operator and the RBAC rules it requires, as well as descriptions of any Custom Resources (CRs) it manages. The CRD files define the schemas for the Custom Resources.
- **Metadata:** This directory contains an annotations.yaml file, which provides metadata about the bundle itself, such as the bundle version, the versions of the APIs provided by the Operator, and the default channel for the Operator. This metadata is used by OLM and OperatorHub to understand the contents of the bundle and how to display information about the Operator.
- **tests/scorecard:** This directory contains configuration for the Operator SDK Scorecard, an internal testing tool provided by the Operator SDK. The Scorecard provides a set of tests that check the Operator for compliance with best practices and certain Kubernetes and Operator SDK conventions. These tests can be customized and extended by the Operator developer to adapt to their specific use case. The results of the Scorecard tests can provide useful feedback during Operator development and can also be used in Continuous Integration/Continuous Deployment (CI/CD) pipelines.

In summary, the bundle directory contains everything needed for the Operator Lifecycle Manager (OLM) to manage the Operator. The manifests subdirectory contains the Operator's deployment specifications and the definitions of its Custom Resources, while the metadata subdirectory contains metadata about the bundle itself.

##### **internal/sub_reconcilers/:**
Reconciliation: The Heart of l5-operator
Here, we'll take an in-depth look at the reconciliation process in l5-operator. We'll explain how it ensures the desired state of Bestie resources and handles updates, deletions, and error scenarios.

#### Database seeding
- **reconcile_seed_data.go** defines a sub-reconciler named DatabaseSeedJobReconciler for seeding data into a Postgres database managed by the Operator.

    Here's a summary of the key components:

    - **DatabaseSeedJobReconciler Struct:** This struct is the main sub-reconciler for the Bestie CR. It includes a client for interacting with the Kubernetes API, a logger, and a runtime.Scheme for converting between different API versions.
    - **Reconcile Method:** This is the core method of the sub-reconciler. It's called as part of the main reconcile loop whenever a Bestie object is created or updated. The method checks if the Postgres instance managed by the Bestie object is ready. If it is, the method creates a Kubernetes Job to seed data into the Postgres database.
    - **applyManifests Function:** This function applies config/resources/bestie-job.yaml file. It's used to create the Job that seeds data into the Postgres database.
    - **createStatusCondition and updateStatusCondition Functions:** These functions update the status of the Bestie object to reflect the state of the database seeding operation.

    In summary, the reconcile_seed_data.go file defines a sub-reconciler for a Kubernetes Operator. This sub-reconciler is responsible for seeding data into a Postgres database managed by the Operator. It waits until the Postgres instance is ready, then creates a Kubernetes Job to seed the data, and updates the status of the Bestie object to reflect the progress of the operation.
 
 #### Managing deployments
 - **reconcile_deployment.go** defines a sub-reconciler named DeploymentReconciler that manages a Kubernetes Deployment for the Operator.

    Here's a summary of the key components:
    - **DeploymentReconciler Struct:** This struct is the main sub-reconciler for the Bestie CR. It includes a client for interacting with the Kubernetes API, a logger, and a runtime.Scheme for converting between different API versions.
    - **Reconcile Method:** This is the core method of the sub-reconciler. It's called as part of the main reconcile loop whenever a Bestie object is created or updated. The method checks if a database seed job has completed successfully. If it has, the method creates or updates a Deployment for the Bestie.
    - **applyDeploymentFromFile Function:** This function reads config/resources/bestie-deploy.yaml, modifies it based on the Bestie's spec, and applies it to the cluster.
    - **createDeploymentCondition and updateDatabaseSeededCondition Functions:** These functions update the status conditions of the Bestie object to reflect the progress of the Deployment creation and database seeding operations.

    In summary, the reconcile_deployment.go file defines a sub-reconciler for the Operator. This sub-reconciler is responsible for creating a Deployment for a Bestie once its database seed job has completed successfully. It updates the status of the Bestie object to reflect the progress of these operations.  

- **reconcile_deployment_updates.go** file defines a sub-reconciler named DeploymentImageReconciler that manages the image version for the operand(the application managed by the Bestie) of the Operator.

    Here's a summary of the key components:
    - **DeploymentImageReconciler Struct:** This struct is the main sub-reconciler for the Bestie CR. It includes a client for interacting with the Kubernetes API, a logger, and a runtime.Scheme for converting between different API versions.
    - **Reconcile Method:** This is the core method of the sub-reconciler. It's called as part of the main reconcile loop whenever a Bestie object is created or updated. The method checks if the Bestie application is running, updates the Deployment's image if necessary, and updates the status of the Bestie object to reflect the progress of these operations.
    - **upgradeOperand Function:** This function upgrades the operand (the application managed by the Bestie) by updating the image of the Deployment if it doesn't match the image specified in the Bestie's spec.
    - **updateDeploymentReadyCondition and updateApplicationStatus Functions:** These functions update the status conditions of the Bestie object to reflect the progress of the Deployment creation and image update operations.

    In summary, the reconcile_deployment_updates.go file defines a sub-reconciler for a Kubernetes Operator. This sub-reconciler is responsible for managing the image version of a Deployment for a Bestie. It updates the Deployment's image if it doesn't match the image specified in the Bestie's spec, and updates the status of the Bestie object to reflect the progress of these operations.

#### Best Practices and Lessons Learned
Building a Kubernetes Operator like the l5-operator is a complex task that requires a solid understanding of Kubernetes concepts and good software design principles. Here are some best practices and lessons learned from the l5-operator:

- **Design for Idempotency:** The Operator pattern is based on the concept of reconciliation loops. Your Operator should be designed to be idempotent, meaning it should achieve the same desired state regardless of how many times it runs. This is an important concept in distributed systems to ensure consistency.

- **Leverage Existing Tools:** The l5-operator uses tools like the Operator SDK and controller-runtime libraries, which provide useful utilities and simplify the process of building an Operator. Using these established tools helps reduce the amount of boilerplate code you have to write and provides a solid foundation for your Operator.

- **Structure for Maintainability:** The l5-operator uses a sub-reconciler pattern, separating different parts of the reconciliation process into separate files and functions. This makes the code easier to understand and maintain. It also allows for better reusability and testing of individual components.

- **Define Clear Custom Resources:** The custom resources that your Operator manages should be well-defined and follow Kubernetes API conventions. The Bestie custom resource in the l5-operator is a good example of a clear and concise custom resource.

- **Implement Robust Error Handling:** Kubernetes is a distributed system, and as such, many things can go wrong. Your Operator should be designed to handle errors gracefully and recover from them when possible.

- **Monitor Your Operator:** The l5-operator includes metrics to monitor the Operator's performance and the state of the resources it manages. Monitoring is important for understanding the health and performance of your Operator and for debugging issues.

- **Use RBAC for Security:** The l5-operator defines specific RBAC roles and bindings to limit the permissions of the Operator to only what it needs to function. This follows the principle of least privilege, enhancing the security of your Kubernetes cluster.

- **Version Your Operator and its CRDs:** As you develop and improve your Operator, it's likely that you'll need to make changes to its custom resources. Versioning your Operator and its CRDs allows you to make these changes in a backward-compatible manner, which is important for maintaining existing deployments of your Operator.

- **Handle Upgrades and Updates Carefully:** The l5-operator includes logic for managing updates to the Bestie's image. When developing an Operator, you should carefully consider how updates to the Operator and its custom resources will be handled to minimize disruption to the managed applications.

#### Conclusion 
We've delving into the details of the Kubernetes Operator pattern, and specifically, dissecting the l5-operator. This exploration underscores the power of Kubernetes Operators to automate and standardize the deployment and lifecycle management of complex applications on Kubernetes.

At the heart of the l5-operator are well-defined Custom Resource Definitions (CRDs) and Custom Controllers. The CRDs extend the Kubernetes API to create Bestie resources. The Custom Controller, comprising various sub-reconcilers, maintains the desired state of these Bestie resources, which includes a Postgres database and a corresponding application Deployment.

This Operator is structured for maintainability, designed for idempotency, and secured with Role-Based Access Control (RBAC). It exemplifies best practices, including the use of existing tools like the Operator SDK and controller-runtime libraries, robust error handling, and effective monitoring.

The l5-operator offers several benefits. It abstracts complex management tasks into simple Kubernetes resources, reduces operational overhead, and provides a consistent and reproducible application management strategy. It's a testament to how Kubernetes Operators can automate operations, free up human operators' time, and reduce the risk of human error.

For those eager to explore further, here are some resources:

[Kubernetes Operators](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/): The official Kubernetes documentation provides a comprehensive introduction to Operators.
[Operator SDK](https://sdk.operatorframework.io/): The Operator SDK is a framework for building Operators, providing tools and libraries that encapsulate best practices.
[Operator Lifecycle Manager](https://olm.operatorframework.io/): OLM helps manage the lifecycle of Operators, from installation through upgrades.
[Go Documentation](https://go.dev/doc/): As the l5-operator is written in Go, the official Go documentation is an excellent resource for understanding the code.
[Custom Resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/): More information about Custom Resources, which are central to the Operator pattern.
Operators are a powerful tool for managing applications on Kubernetes. By mastering their design and implementation, you can unlock the full potential of your Kubernetes platform.

Stay tuned for upcoming posts that dive deeper into Kubernetes operators and related topics!

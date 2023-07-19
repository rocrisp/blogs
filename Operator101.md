<div style="text-align: center;">
    <img src="images/relaxing.jpg" height="200">
</div>

### Even if you're new to software development, creating Kubernetes Operators can be a straightforward process. This guide is designed to make it accessible and easy to understand.

## What is an Operator?
In the context of Kubernetes, an Operator is a method of packaging, deploying, and managing a Kubernetes-native application. Operators leverage **Custom Controllers** and **Custom Resource Definitions** (CRDs) to extend Kubernetes functionality and define application-specific behavior.

## Why Operators?

Deploying applications in Kubernetes or OpenShift often involves understanding complex processes such as deployment, scaling, backup and recovery, and integration. 
This complexity escalates when dealing with stateful applications or intricate integrations. 

However, Operators simplify these challenges by automating these operations, eliminating the need for manual intervention. They handle the automation and management of applications within Kubernetes, taking care of tasks such as deployment, configuration, and ongoing operations. 

By leveraging Operators, the process of application installation is streamlined, with complex configurations and operations handled automatically, thereby reducing reliance on manual efforts.

## Kubernetes Controllers and Custom Controllers
In Kubernetes, controllers are responsible for managing and ensuring the desired state of several resources in the cluster. By using the reconciliation loops which continuously monitor the current state of resources, compare it with the desired state, and take action to reconcile any differences. Controllers help maintain system health, enforce policies, and ensure the proper functioning of applications.

**Custom Controller** extends Kubernetes to support their specific use cases and domain-specific requirements. This flexibility allows for the development of highly specialized and tailored solutions within the Kubernetes ecosystem, providing fine-grained control over the behavior and management of custom resources.  They are typically implemented as custom applications running outside the Kubernetes control plane as standalone processes.


Here are a few examples of Kubernetes controllers and their responsibilities:

- **Deployment Controller**: The deployment controller manages the rollout and updates of applications. It ensures that the desired number of replicas are available, performs rolling updates to deploy new versions, and handles rollbacks if necessary.

- **DaemonSet Controller**: DaemonSet controllers ensure that a specific pod runs on every or specific nodes in the cluster. They are useful for deploying cluster-wide services or agents, such as logging or monitoring agents, that require presence on every node.

- **Job/CronJob Controller**: These controllers manage the execution of batch workloads. The Job controller ensures that a specific task or workload is completed, while the CronJob controller schedules and runs recurring jobs based on a specified schedule.

## Custom Resource Definitions (CRDs)
Custom Resource Definitions (CRDs) serves as the blueprint for custom resource in Kubernetes. It enables Operators to define and manage application-specific resources effortlessly. Customize your Kubernetes applications with ease.

## Operator Hub and Marketplace
Explore the [Operator Hub](https://operatorhub.io/), a centralized platform where you can discover a wide range of pre-built Operators contributed by experts and enthusiasts."

## Building Your Own Operators:
Can't find the Operator you need? No worries! The [Operator Framework](https://operatorframework.io/) empowers you to build your own Kubernetes Operators tailored to your unique requirements.

## Getting Started with Operator SDK:
Start your Operator development journey with Operator SDK. Follow these step-by-step instructions to get up and running quickly. 


#### Step 1: Install Operator SDK
Begin by installing Operator SDK on your development machine. You can find installation instructions specific to your operating system in the [official Operator SDK documentation](https://sdk.operatorframework.io/docs/installation/).

#### Step 2: Create a new Operator project
Use the [`operator-sdk init`](https://sdk.operatorframework.io/docs/building-operators/golang/tutorial/#create-a-new-project) command to initialize a new Operator project. This command will set up the basic directory structure and project files needed for your Operator.

#### Step 3: Define your API and Controller
With Operator SDK, generating the API and controller code for your custom resource is a breeze. Use the `operator-sdk create api --group yourgroup --version v1alpha1 --kind YourKind --resource --controller` command and specify the resource name, group, and version. 

Operator SDK will create the necessary files and code scaffolding for your custom resource and the Controller Code. This code will contain the logic for handling events, reconciling the state, and interacting with Kubernetes resources.

#### Step 4: Implement your Controller logic
Now that you have the basic structure in place, you can customize the generated code to fit your specific requirements. You can [add custom business logic](https://sdk.operatorframework.io/docs/building-operators/golang/tutorial/#implement-the-controller), interact with external systems, or configure advanced features based on your application's needs.

#### Step 5: Build and deploy your Operator
Use the [`make docker-build docker-push`](https://sdk.operatorframework.io/docs/building-operators/golang/tutorial/#configure-the-operators-image-registry) command to build your Operator's container image. This will compile your code and package it into a container image that can be deployed to your Kubernetes cluster.

#### Step 6: Run the Operator
[Deploy your Operator](https://sdk.operatorframework.io/docs/building-operators/golang/tutorial/#run-the-operator) to a Kubernetes cluster and create instances of your custom resource. Validate that your Operator functions as expected and make iterative adjustments based on testing results.

### Information for publishing your Operator on [OperatorHub.io](https://operatorhub.io/contribute)

### Steps of creating, testing, and publishing your operator on OperatorHub.io

![](https://hackmd.io/_uploads/BkJhd6B53.png)


## Conclusion
With tools like Operator SDK, creating a Kubernetes Operator is within reach even for non-developers. 

By following the steps outlined in this guide, you can leverage the power of Operator SDK's scaffolding and automation to build your first Operator. 

Remember to consult the [Official Operator SDK documentation](https://sdk.operatorframework.io/) for more detailed instructions and examples. 

Now, take the plunge and start automating your applications within Kubernetes with your very own Operator. 

Happy coding!

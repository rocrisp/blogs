# Streamlining Enterprise Application Deployment with the **EAP Operator**: A Beginner's Guide

By Rose Crisp

![](https://i.imgur.com/DvDqCN5.jpg)



If you're a Java developer looking for a powerful platform to build highly transactional and scalable applications, you've come to the right place.

In this article, we'll highlights the benefits of JBoss Enterprise Application Platform and provide you with a detailed step-by-step guide on deploying and managing it effortlessly on OpenShift using the EAP Operator.

So, let's dive in and unlock the true potential of your Java applications!


### What is Red Hat JBoss Enterprise Application Platfrom [(JBoss EAP)](https://developers.redhat.com/products/eap/overview)? 

**JBoss EAP** is an open-source platform for building highly transactional and scalable Java applications. By combining Jakarta EE specifications with modern technologies like Eclipse MicroProfile, it enables the transformation of traditional Java EE applications into cloud-native architectures. 

It provides a comprehensive set of tools to develop, deploy, and manage enterprise Java applications in various environments, including on-premise, virtual setups, and private/public/hybrid clouds. 

Based on the widely used open-source project [**WildFly**](https://github.com/wildfly), JBoss EAP offers everything needed to build, run, and manage enterprise Java applications effectively.

### What is the EAP Operator?

Based on the Open Source upstream project, [wildfly-operator](https://github.com/wildfly/wildfly-operator), the **EAP Operator** is a software component that facilitates the deployment and management of the JBoss EAP on container orchestration platforms like OpenShift. The operator acts as an automated management tool, simplifying the process of provisioning, scaling, and updating JBoss EAP applications.

### What is an [Operator](https://www.cncf.io/blog/2022/06/15/kubernetes-operators-what-are-they-some-examples/#:~:text=K8s%20Operators%20are%20controllers%20for,Custom%20Resource%20Definitions%20(CRD).)?

Operators, introduced by the CoreOS Linux team in 2016, simplify application management within Kubernetes. They are software extensions that treat applications as single objects, utilizing custom resources to manage applications and their components. 

Instead of dealing with individual primitives, Operators provide specific adjustments tailored to the application that automate installation, configuration, and ongoing maintenance tasks 

Acting as controllers for packaging, managing, and deploying applications on Kubernetes, Operators utilize Custom Resource Definitions (CRDs) to define desired configurations. They continuously reconcile the actual state of the application with the desired state defined by the CRD, enabling automatic scaling, updates, and restarts. 

Operators extend Kubernetes capabilities, performing complex functions beyond the scope of native Kubernetes operations.

### What are the use cases for using the JBoss EAP operator?

The EAP Operator serves various use cases for organizations looking to deploy and manage multiple instances of the JBoss EAP Java application instances within a Kubernetes cluster.

Let's explore a few scenarios where the EAP Operator shines:

- Simplified Management:

  The EAP Operator can create, configure, manage, and seamlessly upgrade instances of complex stateful applications. It abstracts away the complexities of manual configuration, making management a breeze.

- Efficient Scaling and Updates:

  Effortlessly scale your JBoss EAP Java application instances based on demand. It ensures that your applications can handle increased traffic by automatically provisioning additional resources. Furthermore, the Operator facilitates seamless updates, enabling you to roll out new features and bug fixes without downtime.

- Ensures safe transaction recovery:

  EAP Operator ensures safe transaction recovery in your application cluster by verifying all transactions are completed before scaling down the replicas and marking a pod as clean for termination. The EAP operator uses StatefulSets for the appropriate handling of EJB remoting and transaction recovery processing. The StatefulSet ensures persistent storage and network hostname stability even after pods are restarted.

---

## Installation Steps:
Now, let's walk through the steps to install the EAP Operator on your [OpenShift 4.x Cluster](https://www.redhat.com/en/technologies/cloud-computing/openshift) and deploy your first example application: 


#### Step 1: Create a project

- Login to your OpenShift cluster as the Administrator and go to *Home > Projects*.
- Create a new project and give it a named, such as `wildfly`.

#### Step 2: Install JBoss EAP from OperatorHub

- Navigate to *Operators > OperatorHub*.
- Search for "jboss eap" (provided by Red Hat)and install it.
- Choose "All namespaces on the cluster" as the installation mode(defaults to"openshift-operators" namespace).

#### Step 3: Deploy your first example application

- Switch to the "wildfly" namepsace: `oc project wildfly`.
- Utilize this [Helm Chart](https://github.com/jbossas/eap-charts/tree/main/charts/eap74) to build the JBoss EAP application image using Source-to-Image (S2I) on OpenShift.

  ```
  cat <<EOF > helm.yaml
  build:
    uri: https://github.com/jboss-eap-up-and-running/eap7-getting-started
  deploy:
    enabled: false
  EOF
  
  helm install eap7-app -f ./helm.yaml --repo https://charts.openshift.io redhat-eap74
  ```
- Create the application by applying the following YAML configuration using the `wildflyserver` custom resource:

  ```
  cat <<EOF | oc apply -f -
  apiVersion: wildfly.org/v1alpha1
  kind: WildFlyServer
  metadata:
    name: eap7-app
  spec:
    applicationImage: eap7-app:latest
    replicas: 1
  EOF
  ```

#### Step 4: Access the application website

- The Operator automatically generates routes for you. Simply copy and paste the route into your web browser.
- To view the routes, run: `oc get route`

---

By following these steps, you can successfully install the **EAP Operator** on your OpenShift cluster and deploy your first example application.

Additionally, you can find more examples for experimentation in [this repository](https://github.com/jboss-eap-up-and-running).

The EAP Operator empowers you to harness the full potential of JBoss EAP, simplifying management, enabling efficient scaling, and ensures safe transaction recovery in your application cluster. 

Start exploring the possibilities and unlock the power of your Java applications with the EAP Operator.

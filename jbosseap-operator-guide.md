# Beginner's Guide: Installing JBoss EAP Operator on OpenShift

By Rose Crisp

![](https://i.imgur.com/DvDqCN5.jpg)


### What is [JBoss EAP](https://developers.redhat.com/products/eap/overview)? 

JBoss EAP is an open-source platform for building highly transactional and scalable Java applications. By combining Jakarta EE specifications with modern technologies like Eclipse MicroProfile, it enables the transformation of traditional Java EE applications into cloud-native architectures. JBoss EAP provides a comprehensive set of tools to develop, deploy, and manage enterprise Java applications in various environments, including on-premise, virtual setups, and private/public/hybrid clouds. Based on the widely used open-source project WildFly, JBoss EAP offers everything needed to build, run, and manage enterprise Java applications effectively.
### What is the [JBoss EAP Operator](https://github.com/k8sgpt-ai/k8sgpt-operator/tree/main)?

The JBoss EAP Operator is a Kubernetes-native application designed for deploying and managing JBoss Enterprise Application Platform (EAP) instances within a Kubernetes cluster. It simplifies the management and scaling of JBoss EAP workloads by leveraging Kubernetes capabilities and extending them to support JBoss EAP resources as custom Kubernetes objects. The operator automates provisioning, handles lifecycle management, and integrates with Kubernetes monitoring and logging frameworks. It enables centralized configuration management, supports high availability and scaling, and seamlessly integrates with CI/CD pipelines. The JBoss EAP Operator streamlines the deployment and management of JBoss EAP applications in cloud-native environments, promoting automation, scalability, and efficient resource utilization.

### What is an [Operator](https://www.cncf.io/blog/2022/06/15/kubernetes-operators-what-are-they-some-examples/#:~:text=K8s%20Operators%20are%20controllers%20for,Custom%20Resource%20Definitions%20(CRD).)?



According to the CNCF blog, Operator was introduced in 2016 by the CoreOS Linux team, the Operator concept simplifies application management within Kubernetes. Operators are software extensions that leverage custom resources to manage applications and their components. Rather than dealing with individual primitives, Operators treat applications as single objects, providing specific adjustments tailored to the application.

Operators automate both installation and configuration tasks (Day-1) and ongoing maintenance tasks (Day-2), seamlessly integrating with Kubernetes concepts and APIs. They act as controllers for packaging, managing, and deploying applications on Kubernetes, utilizing Custom Resource Definitions (CRDs) to define desired configurations. Operators continuously reconcile the actual state of the application with the desired state defined by the CRD, enabling automatic scaling, updates, and restarts. Operators extend Kubernetes capabilities, performing more complex functions beyond the scope of native Kubernetes operations.

### What are the use cases for using the JBoss EAP operator operator?

A use case for the JBoss EAP Operator is when an organization wants to deploy and manage multiple instances of JBoss Enterprise Application Platform (EAP) within a Kubernetes cluster. The operator simplifies the management of JBoss EAP workloads by automating provisioning, scaling, and lifecycle management tasks. It provides a standardized and streamlined approach to deploy and manage JBoss EAP applications in a cloud-native environment.


---

## To install the JBoss EAP Operator on [OpenShift 4.x Cluster](https://www.redhat.com/en/technologies/cloud-computing/openshift), follow these steps: 


#### Step 1: Create a project

- Login to your OpenShift cluster as the Administrator and go to *Home > Projects*.
- Create a new project and give it a named, like `wildfly`.

#### Step 2: Install JBoss EAP from OperatorHub

- Navigate to *Operators > OperatorHub*.
- Search for `jboss eap` (provided by Red Hat)and install it.
- Choose`All namespaces on the cluster` as the installation mode(defaults to`openshift-operators` namespace).

#### Step 3: Deploy your first example

- Switch to the "wildfly" namepsace: `oc project wildfly`.
 
- Utilize Helm to build and install the application image using Source-to_Image (S2I)
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

#### Step 4: Access application website

- The Operator automatically generates routes for you. Simply copy and paste the route into your web browser.
- To view the routes, run: `oc get route`

#### By following these steps, you can successfully install the JBoss EAP Operator on your OpenShift cluster and deploy your first example application.

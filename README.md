# DevOps Garage Solution Engineering

Welcome to the Staging environment of the DevOps Garage Solution Engineering team.

:warning: Please note the following observations.
- Only the version of Tekton bundled into the IBM Cloud Pak for Applications (ICP4A) is supported.  If you install a newer or older version of Tekton after the ICPA install, it will override the version that is bundled in ICP4A.
- The [Tekton Webhook Extension](https://github.com/tektoncd/experimental/tree/master/webhooks-extension) currently only supports Github webhooks.
- Kabanero installation includes includes Istio, Knative, Tekton, kAppNav, Appsody, Che).
- ICP4A installation includes Kabanero and Transformation Advisor.


## Deploy the `bluecompute-ce` Cloud Native application on OpenShift
* **Description:**
  - Deploy a Cloud Native demo application to your OpenShift cluster.  
* [Deploy bluecompute-ce](bluecompute-ce/README.md)

## Configure Tekton pipeline for `bluecompute-ce`
* **Description:**
  - Use [Tekton](https://github.com/tektoncd), a Kubernetes Native CICD pipeline resource, to configure pipelines for `bluecompute-ce`.
  - This tutorial will walk you through the installation of the latest Tekton release (v0.8.0 at the time of this post) on an OpenShift cluster.
* [Tekton Pipeline for Web application](bluecompute-ce-tekton-pipelines/web/README.md)
* [Tekton Pipeline for Microservies](bluecompute-ce-tekton-pipelines/Microservice/README.md)

## Configure Kabanero Pipeline for `bluecompute-ce`
* **Description:**
  - Use [Kabanero](https://kabanero.io/), an open source project bringing together foundational open source technologies into a modern microservices-based frameworkm to configure pipelines for `bluecompute-ce`.
  - This tutorial will walk you through the installation of the IBM Cloud Pak for Applications on an OpenShift cluster running on IBM Cloud.
* [Refactor Git repo to use Appsody Stack](bluecompute-ce-kabanero-pipelines/Refactor-Application-Appsody-Stack.md)
* [Kabanero Pipeline for Web application](bluecompute-ce-kabanero-pipelines/README.md)
* [Kabanero Pipeline for Microservices - TBD]

## Configure OpenShift 4x Pipeline for `bluecompute-ce`
* **Description:**
  - Use [OpenShift Pipelines](https://www.openshift.com/learn/topics/pipelines), the Cloud Native CI/CD approach on OpenShift based on Tekton, to configure pipelines for `bluecompute-ce`.
  - This tutorial will walk you through the installation of OpenShift Pipelines on an OpenShift 4x cluster running on VMware.
* [OpenShift Pipelines for Web Application - In Progress](bluecompute-ce-openshift-pipelines/README.md)
* [OpenShift Pipelines for Microservices - TBD]

## Debug Tekton pipelines
* **Description:**
  - Sharing notes on how to debug issues with Tekton Pipelines.  
* [Debugging tips for Tekton Pipelines](faq/README.md)

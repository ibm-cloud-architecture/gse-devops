# DevOps Garage Solution Engineering

Welcome to the Staging environment of the DevOps Garage Solution Engineering team.

:warning: Please note the following observations.
- Only the version of Tekton bundled into the IBM Cloud Pak for Applications (ICP4A) is supported.  If you install a newer or older version of Tekton after the ICPA install, it will override the version that is bundled in ICP4A.
- The [Tekton Webhook Extension](https://github.com/tektoncd/experimental/tree/master/webhooks-extension) currently only supports Github webhooks.
- Kabanero installation includes includes Istio, Knative, Tekton, kAppNav, Appsody, Che).
- ICP4A installation includes Kabanero and Transformation Advisor.
  - ICP4A v3.0.1 bundles Tekton 0.5.2


## Deploy the `bluecompute-ce` Cloud Native application on OpenShift
* [Link](bluecompute-ce/README.md)
* **Description:**
  - Deploy a Cloud Native demo application to your OpenShift cluster.  

## Configure Tekton pipeline for `bluecompute-ce`
* [Link](bluecompute-ce-tekton-pipelines/README.md)
* **Description:**
  - Use [Tekton](https://github.com/tektoncd), a Kubernetes Native CICD pipeline resource, to configure pipelines for `bluecompute-ce`.
  - This tutorial will walk you through the installation of the latest Tekton release (v0.8.0 at the time of this post) on an OpenShift cluster.

## Configure Kabanero Pipeline for `bluecompute-ce`
* [Link](bluecompute-ce-kabanero-pipelines/README.md)
* **Description:**
  - Use [Kabanero](https://kabanero.io/), an open source project bringing together foundational open source technologies into a modern microservices-based frameworkm to configure pipelines for `bluecompute-ce`.
  - This tutorial will walk you through the installation of the IBM Cloud Pak for Applications on an OpenShift cluster running on IBM Cloud.

## Debug Tekton pipelines
* [Link](faq/README.md)
* **Description:**
  - Sharing notes on how to debug issues with Tekton Pipelines.  


# Configure OpenShift Pipeline for App Connect Enterprise Deployments (Work in progress)

This [README](../../cloudpak-for-integration-tekton-pipelines/README.md) provides a working sample of a CICD pipeline involving the following steps:

1. Compile and create a bar based on an ACE Toolkit project.
2. Build an ACE-only server image containg the bar file and push it into the IBM Container Registry.
3. Deploy the newly built image of the ACE-only server + bar into an OpenShift cluster.

This tutorial uses the ACE pipeline created by our fellow GSE GSE colleagues in the Integration squad (https://github.ibm.com/rsundara/cp4i-ace-server).

Reference links:
- App Connect Enterprise Git Repo: https://github.com/ot4i/ace-docker
- ACE Server Helm Chart in the IBM Entitled Registry: https://github.com/IBM/charts/tree/master/entitled/ibm-ace-server-icp4i-prod
- ACE Server Image on Dockerhub: https://hub.docker.com/r/ibmcom/ace


## Requirements:
- Install the Git CLI `git`, Docker CLI `docker`, Kubernetes CLI `kubectl`, OpenShift CLI `oc`, Helm CLI `helm` and `cloudctl` CLI.
- Inventory microservice from the Bluecompute application deployed to an OpenShift / Kubernetes cluster with the service exposed as a route / ingress. This will be required by the sample BAR file. Additional instructions will be provided (TBD)


## Provision OpenShift 4.x cluster
OpenShift clusters can be provisioned on various Cloud providers or on-prem infrastructure.  In this demo, we will be using a Managed OpenShift cluster on IBM Cloud.

The OpenShift 4.2 installation [documentation](https://docs.openshift.com/container-platform/4.2/welcome/index.html) provides installation instructions.  

Our fellow GSE colleagues have also published instructions to install OpenShift 4.x on-prem.
  - [Installing OpenShift 4.x on VMware Virtual Infrastructure](https://github.com/ibm-cloud-architecture/refarch-privatecloud/blob/master/Install_OCP_4.x.md)


## Install OpenShift Pipeline Operator
OpenShift Pipelines is a Kubernetes native solution based on Tekton and provides an opinionated CI/CD experience with tight integration to OpenShift and RedHat developer tools.  It is currently available as a `Developer Preview release` and installation instruction can be found in this [link](https://openshift.github.io/pipelines-docs/docs/0.10.5/index.html).
1. Log on to the OpenShift 4.x Web Console and navigate to Operators > OperatorHub.
2. Search for `pipeline` and click on the `OpenShift Pipelines Operator`.
  ![OpenShift Pipeline Operator](../../static/imgs/openshift-pipeline-operator.png)\
3. A `Show Community Operator` dialog box will pop up, click Continue.
  ![OpenShift Pipeline Community Operator](../../static/imgs/openshift-pipeline-operator-community-msg.png)
4. On the `OpenShift Pipelines Operator` screen, click Install.
  ![Install OpenShift Pipeline Operator](../../static/imgs/openshift-pipeline-operator-install.png)
5. On the `Create Operator Subscription` page, ensure the `All namespaces on the cluster (default)` option is selected and click `Subscribe`.
  ![Create subscription](../../static/imgs/openshift-pipeline-operator-subscription.png)
6. On the `Installed Operators` view, confirm the `OpenShift Pipeline Operator` has been deployed correctly.
  ![Deploy OpenShift Pipeline Operator](../../static/imgs/openshift-pipeline-operator-deploy.png)


## Install Tekton Dashboard
Previous releases are available from [here](https://storage.googleapis.com/tekton-releases/).
1. Install [Tekton Dashboard](https://github.com/tektoncd/dashboard):
```
  # Create a new project called `tekton-pipelines`
  oc new-project tekton-pipelines

  # Install v0.5.3 of Tekton Dashboard as this is compatible with Kubernetes clusters < 1.15.0
  kubectl apply --filename https://github.com/tektoncd/dashboard/releases/download/v0.5.3/openshift-tekton-dashboard-release.yaml  --validate=false

  # Retrieve the Tekton Dashboard Route
  oc get route -n tekton-pipelines
```
![Tekton Dashboard](../../static/imgs/openshift4x-tekton-dashboard.png)


## Configure Namespace for ACE deployment
1. Create a project for the ACE deployment
```
# Create a new project
oc create new-project <project>
````
2. Add Security Context Constraints (SCC) to the default service account
```
# Add SCC Policy
oc adm policy add-scc-to-user anyuid system:serviceaccount:<project>:default
oc adm policy add-scc-to-user privileged system:serviceaccount:<project>:default
```

## Clone the required Git repositories
1. Clone the repository containg the sample ACE Toolkit workspace:
```
git clone git@github.ibm.com:Andrew-Suh/workspace-trigger.git
```
2. Clone the repository containing the sample Tekton yamls:
```
git clone git@github.com:ibm-cloud-architecture/gse-devops.git
```

## Configure Secrets required by the pipeline
The Tekton pipeline will require credentials to clone Github Enterprise repositories, credential to authenticate to the Cloud Pak for Integration and also credentials to push and/or pull images from an image registry.  
Step 2 - 4 can also be accomplished via the Tekton Dashboard.
1. Update `common-settings`:
```
# Modify the Secret/ace-server-secrets.yaml
cd gse-devops/cloudpak-for-integration-tekton-pipelines/Tekton-YAMLs/


# For the common-settings Secret, base64 encode the following values:
    username: <BASE64 ENCODE USERID FOR ACCESSING CLOUD PAK FOR INTEGRATION>
    password: <BASE64 ENCODE PASSWORD FOR ACCESSING CLOUD PAK FOR INTEGRATION>
    url: <<BASE64 ENCODE URL FOR ICP CONSOLE URL>
    cloudType: <BASE64 ENCODE ("ibmcloud" or "onprem")>
    offlineInstall: <BASE64 ENCODE TRUE OR FALSE>
    fileStorage: <BASE64 ENCODE "nfs" or "csi-cephfs" or "ibmc-file-gold" or "any RWX storage provider">
```
2. Update `git-secret`:
```
# For the git-secret, base64 encode the following values:
    password: <BASE64_ENCODE(API_KEY)>
    username: <BASE64_ENCODE(GIT_USER_NAME)>
```
3. Update `docker-secret`:
```
    username: <USERID_USED_TO_LOGIN_TO_OPENSHIFT_CLUSTER>
    password: <PASSWORD_TO_LOGIN_TO_OPENSHIFT_CLUSTER>
```
4. Create the Secrets in the cluster:
```
oc apply -f Secret/ace-server-secret.yaml -n <project>
```
5. Update the `pipeline` service account with the `git-secret` and `docker-secret`
```
# Add "docker-secret" and "git-secret" to "pipeline" service account:
oc edit sa pipeline -n <project>

# Under the "secrets" section, add entries referencing the git and docker secrets

secrets:
- name: pipeline-token-xxxxxx
- name: pipeline-dockerxxxxxx
- name: docker-secret
- name: git-secret
```


## Configure Tekton PipelineResources
The Tekton pipeline will require PipelineResources as input and output in order to execute.  For example, for the ACE server deployment pipeline, the input would be referencing a Git repository containing the ACE Toolkit workspace artifacts.  Similarly, one of the outputs expected when the pipeline is complete is an image of the ACE Server with the newly built BAR file.
1. Update `cp4i-build-image` with the URL of the expected ACE server image to be created.
```
value: image-registry.openshift-image-registry.svc:5000/<namespace>/<image name>

# If using a private registry, such as the IBM Container Registry, the URL would be:
value: us.icr.io/<namespace>/<repository>:<tag>
```
2. Update `cp4i-ace-server-source` with the URL of the target Git repo
```
value: <Github repo URL>

# For this tutorial, we will be using the following repoitory
value: https://github.ibm.com/Andrew-Suh/workspace-trigger
```
3. Create the PipelineResources in the cluster:
```
oc apply -f PipelineResources/ace-server-resources.yaml -n <project>
```


## Create ACE Server pipeline
1. Create the Tekton Pipeline and Task resources
```
oc apply -f Pipelines-Tasks/
```
2. From the Tekton Dashboard, validate that the ACE Server Pipelines and Tasks appears in the <project> namespace.


## Manually Trigger the ACE Tekton Pipeline from the Tekton dashboard
1. From the Tekton dashboard, select PipelineRuns and click the `Create` button.
2. In the Create PipelineRun window, provide the following values and click Create.
```
Namespace: <project>
Pipeline: ace-server-deploy
PipelineResources:
  git-source: cp4i-ace-server-source
  image: cp4i-build-image
Params:
  project: <Name of ACE Toolkit project being built>
  buildversion: Tag of the image being built
  env: dev
  production: Specifies if the deployment is production-like with High Availability enabled. It is set to false by default.
Service Account: pipeline
```
3. Click on the link to the newly executed PipelineRun and monitor the status.  You can also run `oc get pods -w` in the <project> to view the status of the pods.


## Configure Tekton Github Webhook:
TBD

## Validate webhook by pushing in a change to the web application:
TBD

## How to create a new release and update the Kabanero CR

To create a new release and update the Kabanero CR you'll have to be logged in to an Openshift Cluster
with Kabanero and Tekton installed.

Fork this repository https://github.com/ibm-cloud-architecture/devops-demo-kabanero-pipelines

    git clone https://github.com/your-organization-name/devops-demo-kabanero-pipelines
    
    cd ./devops-demo-kabanero-pipelines

** Ensure you do a git clone `https` instead of `ssh` since there is currently a protocol issue.  **

Take a look at the folder `pipelines/incubator`. This is where your pipelines exist, you can either update or modify these pipelines for your own custom needs. If you decide to add your own pipelines, use the `sample-template-structure` folder.

We will be using a script to automate a few steps go to the root of the repository and run `./run.sh`
``` bash
    ===========================================================================

    ======================== AUTOMATOR SCRIPT =================================

    ===========================================================================

      Do you want to
        1) Set up environment, containerzied pipelines and release them to a registry?
        2) Add, commit and push your latest changes to github?
        3) Create a git release for your pipelines?
        4) Upload an asset to a git release version?
        5) Update the Kabanero CR custom resource with a release?
        6) Add a stable pipeline release version to the Kabanero custom resource?
        enter a number > 

```

We will need to zip the pipelines into a tar file in which and upload it to a release but before that we need to create a release. Type `1` onto the terminal and you should see the following output: 

    .
    .
    .
    Asset name: mcm-pipelines/bindings/nodejs-mcm-pl-pullrequest-binding.yaml
    Asset name: manifest.yaml
    --- Created kabanero-pipelines.tar.gz

Once we have this file we can then move forward on creating the release for your repository.
Make any changes if neccessary and push your changes.

You can create the release manually by following the steps here https://help.github.com/en/github/administering-a-repository/managing-releases-in-a-repository

You can also use the `.run.sh` to do it automatically for example: 
``` bash
      Do you want to
        1) Set up environment, containerzied pipelines and release them to a registry?
        2) Add, commit and push your latest changes to github?
        3) Create a git release for your pipelines?
        4) Upload an asset to a git release version?
        5) Update the Kabanero CR custom resource with a release?
        6) Add a stable pipeline release version to the Kabanero custom resource?
        enter a number > 3
    **************************************************************************

    ================ BEGIN CREATING RELEASE ===================================

    **************************************************************************

    Enter Release Version i.e v1.0 : v1.1
    Enter description of release creating a release!
    Create release v1.1 for repo: oiricaud/devops-demo-kabanero-pipelines branch: master
    {
      "url": "https://api.github.com/repos/oiricaud/devops-demo-kabanero-pipelines/releases/25968362",
      "assets_url": "https://api.github.com/repos/oiricaud/devops-demo-kabanero-pipelines/releases/25968362/assets",
       .
       .
       .
      },
      "prerelease": false,
      "created_at": "2020-04-28T02:13:12Z",
      "published_at": "2020-04-28T15:14:57Z",
      "assets": [

      ],
      "tarball_url": "https://api.github.com/repos/oiricaud/devops-demo-kabanero-pipelines/tarball/v1.1",
      "zipball_url": "https://api.github.com/repos/oiricaud/devops-demo-kabanero-pipelines/zipball/v1.1",
      "body": "creating a release!"
    }
    **************************************************************************

    ================ FINISHED CREATING RELEASE ================================

    **************************************************************************
```

Now that you have a release in your repository, you must upload the `ci/assets/default-kabanero-pipelines.tar.gz` file to the release. You can again, do this manually or you can do this from the `./run.sh` script by typing `4` for example: 

``` bash
     Do you want to
        1) Set up environment, containerzied pipelines and release them to a registry?
        2) Add, commit and push your latest changes to github?
        3) Create a git release for your pipelines?
        4) Upload an asset to a git release version?
        5) Update the Kabanero CR custom resource with a release?
        6) Add a stable pipeline release version to the Kabanero custom resource?
        enter a number > 4
    **************************************************************************

    ================ BEGIN UPLOADING ASSET ====================================

    **************************************************************************

    Upload asset to what version? i.e v1.0 : v71.0
    {"url":"https://api.github.com/repos/oiricaud/pipelines/releases/assets/20240735","id":20240735,"node_id":"MDEyOlJlbGVhc2VBc3NldDIwMjQwNzM1","name":"default-kabanero-pipelines.tar.gz","label":"","uploader":{"login":"oiricaud","id":11867058,"node_id":"MDQ6VXNlcjExODY3MDU4","avatar_url":"https://avatars2.githubusercontent.com/u/11867058?v=4","gravatar_id":"","url":"https://api.github.com/users/oiricaud","html_url":"https://github.com/oiricaud","followers_url":"https://api.github.com/users/oiricaud/followers","following_url":"https://api.github.com/users/oiricaud/following{/other_user}","gists_url":"https://api.github.com/users/oiricaud/gists{/gist_id}","starred_url":"https://api.github.com/users/oiricaud/starred{/owner}{/repo}","subscriptions_url":"https://api.github.com/users/oiricaud/subscriptions","organizations_url":"https://api.github.com/users/oiricaud/orgs","repos_url":"https://api.github.com/users/oiricaud/repos","events_url":"https://api.github.com/users/oiricaud/events{/privacy}","received_events_url":"https://api.github.com/users/oiricaud/received_events","type":"User","site_admin":false},"content_type":"application/octet-stream","state":"uploaded","size":6009,"download_count":0,"created_at":"2020-04-28T20:14:45Z","updated_at":"2020-04-28T20:14:45Z","browser_download_url":"https://github.com/oiricaud/pipelines/releases/download/v71.0/default-kabanero-pipelines.tar.gz"}
    
    **************************************************************************

    ================ FINISHED UPLOADING ASSET =================================

    **************************************************************************
```

Now go to your repo -> releases and you should see a release with your tar file attached. We have to now update
the kabanero Custom Resource to point to this release.

Run `oc get kabanero kabanero -o yaml` and you should get an output similar to this

``` yaml
  apiVersion: kabanero.io/v1alpha2
  kind: Kabanero
  name: kabanero
  namespace: kabanero
spec:
  admissionControllerWebhook: {}
  .
  .
  .
  github:
    organization: oiricaud
    teams:
    - collection-admins
    - admins
  stacks:
    pipelines:
    - https:
        url: https://github.com/kabanero-io/kabanero-pipelines/releases/download/0.6.1/default-kabanero-pipelines.tar.gz
      id: default
      sha256: 64aee2805d36127c2f1e0e5f0fc6fdae5cef19360c1bb506137584f3bd0988cc
    repositories:
    - https:
        url: https://github.com/kabanero-io/kabanero-stack-hub/releases/download/0.6.3/kabanero-stack-hub-index.yaml
  .
  .
  .
```

Edit the $ORG, $REPO_NAME and $RELEASE_VERSION and also generate the sha256 by running 
`sha256-value=shasum -a 256 default-kabanero-pipelines.tar.gz`

``` yaml
    - https:
        url: https://github.com/$ORG/$REPO_NAME/releases/download/$RELEASE_VERSION/default-kabanero-pipelines.tar.gz
      id: default
      sha256: $sha256-value
```

Add this to the Kabanero custom resource.

You can also watch the script in action https://asciinema.org/a/315675


## Uninstall OpenShift Pipelines:
1. Log on to the OpenShift 4.x Web Console and navigate to Operators > Installed Operators.
2. On the Installed Operators screen, click `OpenShift Pipelines Operators`.
3. On the Operator Details screen, click Actions > Uninstall Operator.
  ![OpenShift Pipeline Operator](../../static/imgs/openshift-pipeline-operator-uninstall.png)


## Uninstall Tekton components:
```
# Delete Tekton Dashboard
kubectl delete --filename https://github.com/tektoncd/dashboard/previous/v0.5.3/openshift-tekton-dashboard-release.yaml
```

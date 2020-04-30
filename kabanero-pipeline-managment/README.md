## How to create a new release and update the Kabanero CR

To create a new release and update the Kabanero CR you'll have to be logged in to an Openshift Cluster
with Kabanero and Tekton installed.

Fork this repository https://github.com/ibm-cloud-architecture/devops-demo-kabanero-pipelines

    git clone https://github.com/your-organization-name/devops-demo-kabanero-pipelines
    
    cd ./devops-demo-kabanero-pipelines

** Ensure you do a git clone `https` instead of `ssh` since there is currently a protocol issue.  **

Take a look at the folder `pipelines/incubator`. This is where your pipelines exist, you can either update or modify these pipelines for your own custom needs. If you decide to add your own pipelines, use the `sample-template-structure` folder.

We will be using a script to automate a few steps. Go to the root of the repository and run `./run.sh`
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

We will need to zip the pipelines into a tar file and generate a sha256 for each Tekton type and upload this zip file to a release. Type `1` onto the terminal to generate your zip file you should see the following output: 

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
Refresh your tekton webpage and you should see your new pipelines added.

If you don't see your pipelines or some are missing run

``` bash
oc get pods | grep "kabanero"

    kabanero-cli-6bd6bcd794-cggr8                                     1/1       Running            0          6d2h
    kabanero-operator-5fb946d758-7h6n5                                1/1       Running            0          6d2h
    kabanero-operator-admission-webhook-6b9d4d777f-77fhb              1/1       Running            0          6d23h
    kabanero-operator-collection-controller-5bfcb8889d-84pmc          1/1       Running            0          6d2h
    kabanero-operator-stack-controller-c7bcfdf96-7pnsz                1/1       Running            0          6d2h

oc logs kabanero-operator-stack-controller-c7bcfdf96-7pnsz
.
.
.
```

You can also watch the script in action https://asciinema.org/a/315675


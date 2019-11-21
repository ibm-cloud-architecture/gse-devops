
# FAQ

## Troubleshooting tips for Tekton Pipelines
- **Tip 1: Install the required CLIs**
    - Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) CLI.
    - Install [tkn](https://github.com/tektoncd/cli) CLI.
    - Install [oc](https://docs.openshift.com/enterprise/3.2/cli_reference/get_started_cli.html) CLI if you are using an OpenShift cluster.


- **Tip 2: Confirm the credentials / token to access the registry is valid**
    - Verify that you are able to pull images from your private registry to your local machine.
      ```
      # Log in to the registry
      docker login -u <user> -p <password> <Registry URL>
      # For IBM Container Registry
      docker login -u iamapikey -p <token> <Registry URL>

      # Pull an arbitrary image such as Busybox
      docker pull <Registry URL>/<namespace>/<image>:<tag>
      ```
    - Verify that you are able to push images to your private registry from your local machine.
      ```
      # Log in to the registry
      docker login -u <user> -p <password> <Registry URL>
      # For IBM Container Registry
      docker login -u iamapikey -p <token> <Registry URL>

      # Pull an arbitrary image such as Busybox
      docker pull busybox

      # Tag the image
      docker tag busybox <Registry URL>/<namespace>/busybox

      # Push the image
      docker push <Registry URL>/<namespace>/busybox
      ```


- **Tip 3: Check that the service account image has a push and pull secret**
    - Confirm that the traget service account being used has an associated image pull and push secret.
      ```
      # Check the secrets associated to a service account
      kubectl describe serviceaccount <service account name> -n <namespace>
      ```


- **Tip 4: Check to make sure all variable substituion is using $() as of Tekton Pipeline v0.7.0**
    - In Tekton Pipeline v0.7.0 [release](https://github.com/tektoncd/pipeline/releases/tag/v0.7.0), there is no backward compatibility support for usage of ${} for variable substituion.  Only $() is supported.  

## Common Tekton Pipeline failures
  - **Scenario 1: Service account with insufficient permissions**
    - Sample log
    ```
      kubectl -n kabanero logs nodejs-manual-bluecompute-web-pr-hghwf-build-task-dv647-pod-82ad24 -c step-validate-collection-is-active
      Error from server (Forbidden): collections.kabanero.io is forbidden: User "system:serviceaccount:kabanero:default" cannot list collections.kabanero.io in the namespace "kabanero": no RBAC policy matched
      The Kabanero collection contained in kabanero/nodejs:0.2 is not active on this system and cannot be built.
    ```
  - **Scenario 2: Variable substitution failures**
    - Sample log
    ```
      kubectl -n kabanero logs nodejs-manual-bluecompute-web-pr-t4rr5-build-task-shpjp-pod-95714e -c step-build-bud
      error reading info about "/workspace/${inputs.params.pathToContext}/${inputs.params.pathToDockerFile}": stat /workspace/${inputs.params.pathToContext}/${inputs.params.pathToDockerFile}: no such file or directory
    ```
  - **Scenario 3: Service account with no image push secret**
    - Sample log
    ```
      kubectl -n bluecompute logs bluecompute-web-pr-48hnk-build-task-7dv6s-pod-8ea6fa -c step-build-push
      error checking push permissions -- make sure you entered the correct tag name, and that you are authenticated correctly, and try again: checking push permission for "us.icr.io/jhovany/bluecompute-web:0.6.1": creating push check transport for us.icr.io failed: unsupported status code 401
    ```
  - **Scenario 4: Service account with no image pull secret**
    - Sample log
    ```
      Warning  Failed     7m20s (x4 over 8m54s)   kubelet, 10.114.28.165  Failed to pull image "us.icr.io/jhovany/bluecompute-web:0.6.1": rpc error: code = Unknown desc = unable to retrieve auth token: invalid username/password
      Warning  Failed     7m20s (x4 over 8m54s)   kubelet, 10.114.28.165  Error: ErrImagePull
    ```  

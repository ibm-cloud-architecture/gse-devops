
# Refactor Bluecompute web and microservices to use out of the box Appsody Stacks

This [README](../bluecompute-ce-kabanero-pipelines/Refactor-Application-Appsody-Stack.md) will cover how to refactor the the `Bluecompute` web application and microservice repoistories to use out of the box Appsody Stacks.

## Refactor Bluecompute Web application to use the kabanero/nodejs Stack
1. Fork the [bluecompute-web](https://github.com/ibm-cloud-architecture/refarch-cloudnative-bluecompute-web/) repository.
2. Clone the forked repository.
    ```
    git clone git@github.com:<Github Org>/refarch-cloudnative-bluecompute-web.git
    ```
2. Switch to the `spring` branch
    ```
    git checkout spring
    ```
3. Copy contents of `/StoreWebApp` into the parent directory.
    ```
    cp -R StoreWebApp/ .
    ```
4. Delete `/StoreWebApp` directory.
    ```
    rm -rf StoreWebApp
    ```    
5. Follow instructions from Appsody on how to [initalize an existing project](https://appsody.dev/docs/using-appsody/initializing-project#configuring-an-existing-project) with an existing Appsody Stack.
    1. List the available stacks and determine which stack resembles your project.
        appsody list
    2. For this web application, we will be using the nodejs stack. Run `appsody init <stack> none` to initialize your project.  The `.appsody-config.yaml` file will be generated in your project.  
        ```
        appsody init kabanero/nodejs none
        ```
       - Sample output:
        ```
        appsody init kabanero/nodejs none
        No stack requirements set. Skipping...
        Running appsody init...
        Downloading nodejs template project from https://github.com/kabanero-io/collections/releases/download/0.2.0/incubator.nodejs.v0.2.5.templates.simple.tar.gz
        Download complete. Do not unzip the template project. Only extracting .appsody-config.yaml file from /Users/hollisc/Documents/DevOps-GSE/DevOps/CICD-Use-Cases/OpenShift42-CSP/temp/refarch-cloudnative-bluecompute-web/nodejs.tar.gz
        Setting up the development environment
        Your Appsody project name has been set to refarch-cloudnative-bluecompute-web
        Pulling docker image docker.io/kabanero/nodejs:0.2
        Running command: docker pull docker.io/kabanero/nodejs:0.2
        0.2: Pulling from kabanero/nodejs
        340ff6d7f58c: Already exists
        ..
        ..
        ..
        11a4a55d37eb: Pull complete
        Digest: sha256:3b3a914219e643bb982a8c19be92137efee5b5b579e3e70cc4638c9ac0d3e93a
        Status: Downloaded newer image for kabanero/nodejs:0.2
        [Warning] The stack image does not contain APPSODY_PROJECT_DIR. Using /project
        Running command: docker run --rm --entrypoint /bin/bash docker.io/kabanero/nodejs:0.2 -c find /project -type f -name .appsody-init.sh
        Successfully initialized Appsody project with the kabanero/nodejs stack and no template.
        ```
    3. Optional: To change the name of your Appsody project, edit `project-name` property in the `.appsody-config.yaml.`.  For our project, we will rename it to `web`.
    4. Generate AppsodyApplication deployment manfist for your project.
        ```
        appsody deploy --generate-only
        ```
       - Sample output:
        ```
        appsody deploy --generate-only
        Extracting project from development environment
        Pulling docker image docker.io/kabanero/nodejs:0.2
        Running command: docker pull docker.io/kabanero/nodejs:0.2
        0.2: Pulling from kabanero/nodejs
        Digest: sha256:3b3a914219e643bb982a8c19be92137efee5b5b579e3e70cc4638c9ac0d3e93a
        Status: Image is up to date for kabanero/nodejs:0.2
        [Warning] The stack image does not contain APPSODY_PROJECT_DIR. Using /project
        Running command: docker create --name web-extract -v /Users/hollisc/Documents/DevOps-GSE/DevOps/CICD-Use-Cases/OpenShift42-CSP/temp/refarch-cloudnative-bluecompute-web/:/project/user-app index.docker.io/kabanero/nodejs:0.2
        Running command: docker cp web-extract:/project /Users/hollisc/.appsody/extract/web
        Running command: docker rm web-extract -f
        Project extracted to /Users/hollisc/.appsody/extract/web
        Running command: docker build -t dev.local/web --label description=This image contains the Kabanero development stack for the Nodejs collection
        ..
        ..
        ..
        [Docker] Successfully built 9f1bad2cc6c5
        [Docker] Successfully tagged dev.local/web:latest
        Built docker image dev.local/web
        Running command: docker create --name web-extract index.docker.io/kabanero/nodejs:0.2
        Running command: docker cp web-extract:/config/app-deploy.yaml /Users/hollisc/Documents/DevOps-GSE/DevOps/CICD-Use-Cases/OpenShift42-CSP/temp/refarch-cloudnative-bluecompute-web/app-deploy.yaml
        Running command: docker rm web-extract -f
        Created deployment manifest: /Users/hollisc/Documents/DevOps-GSE/DevOps/CICD-Use-Cases/OpenShift42-CSP/temp/refarch-cloudnative-bluecompute-web/app-deploy.yaml
        ```
6. Replace the contents of `/config/default.json` with data from [link](https://github.com/hollisc/refarch-cloudnative-bluecompute-web-appsody/blob/master/config/default.json).
7. Update AppsodyApplication yaml (deployment manifest).  The value of `pullSecret` will contain the image pull secret and this will be assigned to a specified service account via `serviceAccountName` property or a service account with the same name as the Appsody project will be created and set with the image pull secret.
    - Set spec.server.port to `80`
    - Set spec.server.type to `ClusterIP`
    - Add spec.pullSecret with value of `ibm-cr-pull-secret`
8. Commit and push the changes to your own personal Github repository.
    ```
    git commit -am "Refactor repository to use Appsody nodejs stack"
    git push origin spring
    ```

## Refactor Bluecompute TBD microservice to use the kabanero/java-spring-boot2 Stack
TBD

# Deploying Springboot Application to Kubernetes on IBM Cloud via Toolchain

## 1. Fork GitHub repository
1. In GitHub, fork this [example repo](https://github.com/jushchuk/santas-grotto).
2. There are two branches that are important: `springboot` and `postgre`. In the `springboot` branch, you will have to make small modifications which we will cover in a later step.

## 2. Create Database Toolchain

1. Inside IBM Cloud account, go to [toolchains](https://cloud.ibm.com/devops/toolchains?env_id=ibm:yp:eu-gb)
2. Make sure Resource Group is your individual `group-<x>` and Location is London. Click `Create toolchain`
3. Select `Develop a Kubernetes App`
4. In the options, change `Select a source provider` to `GitHub`. Follow instructions to link GitHub account that contains the forked repository.
5. Change `Clone` to `Existing` and select the right repository from the dropdown list.
6. Click on `Delivery Pipeline` (clicking create now will bring you to the same tab). Add new IBM Cloud API key by clicking `New +` (note this is a secret credential, DO NOT SHARE). You will see a popup, default values are ok, but it is encouraged to add a description to the API key with something related to your project.
7. Values should be automatically populated. **IMPORTANT**: you must provide the correct `Cluster namespace`, do not use the default `prod`. Use the namespace that follows this pattern: `group-<lowercaseprojectname>`. 
8. Update `Toochain name` and `App name` to include your project name as part of them and the phrase `database` or `postgre` etc.. I would recommend you use the same phrase for both for simplicity. **Remember the `Toochain name` for future step**.
9. Click `Create`.

## 3. Modifying Database toolchain

### 3.1 Modifying Build stage
1. Once the toolchain is created, you should see a page with multiple tiles. Click the `Delivery Pipeline` tile to adjust some setting before first deployment.
2. Click on the settings gear of `Build` tile then `Configure Stage`.
5.  Under `Input` tab, switch the `Branch` to `postgre`.
6. Click `Save` at the bottom.

### 3.2 Modifying Containerize stage
1. Click on the settings gear of `Containerize` tile then `Configure Stage`.
2. Under `Jobs` tab, switch the `Tester Type` to `Simple`.
3. Click `Save` at the bottom.

### 3.3 Modifying Deploy stage
1. Click on the settings gear of `Deploy` tile then `Configure Stage`.
2. Under the `Jobs` tab, inside `Deploy script`, replace what is in the textbox with
```
source ./scripts/check_and_deploy_kubectl.sh
```
3. Click `Save` at the bottom.

At this point, you should click the Play button in the Build tile to restart the toolchain and ensure your modifications are applied. Next step is to create the toolchain to deploy the Springboot app itself.

## 4. Modify app branch of GitHub repo

An important step is to connect the database with the app. We can do this by modifying a field in the `Dockerfile` in the `springboot` branch. The line
```
ENV PG_HOST=......
```
should be replaced with
```
ENV PG_HOST=<your-database-toolchain-name>
```
where `<your-database-toolchain-name>` is what you remembered from step 2.8. This is the hostname that the app container will use to find the database container.

## 5. Create Springboot Toolchain
A toolchain also needs to be created for the application. Many of the steps will be similar, but with some important differences.

1. Inside IBM Cloud account, go to [toolchains](https://cloud.ibm.com/devops/toolchains?env_id=ibm:yp:eu-gb)
2. Make sure Resource Group is your individual `group-<x>` and Location is London. Click `Create toolchain`
3. Select `Develop a Kubernetes App`
4. In the options, change `Select a source provider` to `GitHub`. Your GitHub account should be linked from prior steps.
5. Change `Clone` to `Existing` and select the right repository from the dropdown list.
6. Click on `Delivery Pipeline` (clicking create now will bring you to the same tab). Add new IBM Cloud API key by clicking `New +` (note this is a secret credential, DO NOT SHARE). You will see a popup, default values are ok, but it is encouraged to add a description to the API key with something related to your project.
7. Values should be automatically populated. **IMPORTANT**: you must provide the correct `Cluster namespace`, do not use the default `prod`. Use the namespace that follows this pattern: `group-<lowercaseprojectname>`. 
8. Update `Toochain name` and `App name` to include your project name as part of them.
9. Click `Create`.

## 6. Modifying Springboot toolchain

### 6.1 Modifying Build stage
1. Once the toolchain is created, you should see a page with multiple tiles. Click the `Delivery Pipeline` tile to adjust some setting before first deployment.
2. Click on the settings gear of `Build` tile then `Configure Stage`.
3. Under `Jobs` tab, switch the `Builder Type` to `Maven` (since this Springboot example uses that).
4. Inside `Build script`, replace what is in the textbox with
```
#!/bin/bash
mvn -B package -DskipTests=True
```
5.  Under `Input` tab, switch the `Branch` to `Springboot`.
6. Click `Save` at the bottom.

### 6.2 Modifying Containerize stage
1. Click on the settings gear of `Containerize` tile then `Configure Stage`.
2. Under `Jobs` tab, switch the `Tester Type` to `Simple`.
3. Click `Save` at the bottom.

### 6.3 Modifying Deploy stage
1. Click on the settings gear of `Deploy` tile then `Configure Stage`.
2. Under the `Jobs` tab, inside `Deploy script`, replace what is in the textbox with
```
source ./scripts/check_and_deploy_kubectl.sh
```
3. Click `Save` at the bottom.

At this point, you should click the Play button in the Build tile to restart the toolchain and ensure your modifications are applied.

## 7. View app

1. In IBM Cloud, go to [Clusters](https://cloud.ibm.com/kubernetes/clusters) and select `classroom-eu-gb-1-bx2.4x16`
2. Click `Kubernetes Dashboard` (blue button in top right). This will open a new tab.
3. In the top right, select your namespace with the format `group-<lowercaseprojectname>`.
4. Click on `Services` (row on left pane). You should see two services.
5. Click on the `External Endpoint` link for the service that corresponds to the springboot app and not the database.
6. You should see the Santa's Grotto app! The default `username/password` combo is `admin/zxc`.

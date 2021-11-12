# Deploying Springboot Application to Kubernetes on IBM Cloud via Toolchain

## 1. Fork GitHub repository
1. In GitHub, fork this [example repo](https://github.com/jushchuk/santas-grotto).
2. There are two branches that are important: `springboot` and `postgre`. In each branch, you will have to make small modifications

## 2. Create Database Toolchain

1. Inside IBM Cloud account, go to [toolchains](https://cloud.ibm.com/devops/toolchains?env_id=ibm:yp:eu-gb)
2. Make sure Resource Group is your individual group-<x> and Location is London. Click `Create toolchain`
3. Select `Develop a Kubernetes App`
4. In the options, change `Select a source provider` to `GitHub`. `Toochain name` and later `App name` should have your project name as part of them and the phrase `database` or `postgre` etc.. **Remember the `App name` for future step**. Other options can be left as defaults.
5. Click on `Delivery Pipeline` (clicking create now will bring you to the same tab). Add new IBM Cloud API key by clicking `New +` (note this is a secret credential, DO NOT SHARE). You will see a popup, default values are ok, but it is encouraged to add a description to the API key with something related to your project.
6. Values should be automatically populated. **IMPORTANT**: you must provide the correct `Cluster namespace`, do not use the default `prod`. Use the namespace that follows this pattern: `group-<lowercaseprojectname>`. Then click `Create`.

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
PG_HOST=santas-grotto-postgre-toolchain-202111020943
```
should be replaced with
```
ENV PG_HOST=<your-database-app-name>
```
where `<your-database-app-name>` is what you remembered from step 2.4.

## 5. Create Springboot Toolchain
A toolchain also needs to be created for the application. Many of the steps will be similar, but with some important differences.

1. Inside IBM Cloud account, go to [toolchains](https://cloud.ibm.com/devops/toolchains?env_id=ibm:yp:eu-gb)
2. Make sure Resource Group is your individual group-<x> and Location is London. Click `Create toolchain`
3. Select `Develop a Kubernetes App`
4. In the options, change `Select a source provider` to `GitHub`. `Toochain name` and later `App name` should have your project name as part of them. Other options can be left as defaults.
5. Click on `Delivery Pipeline` (clicking create now will bring you to the same tab). Add new IBM Cloud API key by clicking `New +` (note this is a secret credential, DO NOT SHARE). You will see a popup, default values are ok, but it is encouraged to add a description to the API key with something related to your project.
6. Values should be automatically populated. **IMPORTANT**: you must provide the correct `Cluster namespace`, do not use the default `prod`. Use the namespace that follows this pattern: `group-<lowercaseprojectname>`. Then click `Create`.

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

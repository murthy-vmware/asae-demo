Instructions needed to run a simple asa-e hello world application. 

These instructions are a subset of the instructions at https://github.com/Azure-Samples/acme-fitness-store

## 1. Pre-requisites

In order to deploy a Java app to cloud, you need
an Azure subscription. If you do not already have an Azure
subscription, you can activate your
[MSDN subscriber benefits](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)
or sign up for a
[free Azure account]((https://azure.microsoft.com/free/)).

In addition, you will need the following:

| [Azure CLI version 2.17.1 or higher](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)
| [Git](https://git-scm.com/)
| [`jq` utility](https://stedolan.github.io/jq/download/)
|

## 2. Install the Azure CLI extension

Install the Azure Spring Apps extension for the Azure CLI using the following command

```shell
az extension add --name spring
```

## 3. Prepare your environment for deployments

```shell
cp ./setup-env-variables-template.sh ./setup-env-variables.sh
```

Open `./setup-env-variables.sh` and update the following information:

```shell
export SUBSCRIPTION=subscription-id                 # replace it with your subscription-id
export RESOURCE_GROUP=resource-group-name           # existing resource group or one that will be created in next steps
export SPRING_APPS_SERVICE=azure-spring-apps-name   # name of the service that will be created in the next steps
export REGION=region-name                           # choose a region with Enterprise tier support
export APP_NAME=app-name
```

Then, set the environment:

```shell
source ./setup-env-variables.sh
```

## 4. Login to Azure

Login to the Azure CLI and choose your active subscription. 

```shell
az login
az account list -o table
az account set --subscription ${SUBSCRIPTION}
```

Accept the legal terms and privacy statements for the Enterprise tier.

```shell
az provider register --namespace Microsoft.SaaS
az term accept --publisher vmware-inc --product azure-spring-cloud-vmware-tanzu-2 --plan asa-ent-hr-mtr
```

## 5. Create Azure Spring Apps service instance

Create a resource group to contain your Azure Spring Apps service.

> Note: This step can be skipped if using an existing resource group

```shell
az group create --name ${RESOURCE_GROUP} \
    --location ${REGION}
```

Create an instance of Azure Spring Apps Enterprise.

```shell
az spring create --name ${SPRING_APPS_SERVICE} \
    --resource-group ${RESOURCE_GROUP} \
    --location ${REGION} \
    --sku Enterprise \
    --enable-application-configuration-service \
    --enable-service-registry \
    --enable-gateway \
    --enable-api-portal \
    --build-pool-size S2 
```

> Note: The service instance will take around 10-15 minutes to deploy.

Set your default resource group name and cluster name using the following commands:

```shell
az configure --defaults \
    group=${RESOURCE_GROUP} \
    location=${REGION} \
    spring=${SPRING_APPS_SERVICE}
```

## Create App
```shell
az spring app create -n ${APP_NAME}
```
## Test hello-world app locally
Perform the below steps to build the app locally

```bash
cd hello-world
./mvnw spring-boot:run
```

Test the application locally, from a different terminal window, e.g.
```bash
curl http://localhost:8080/
```

Also, you can test the application using a browser, e.g. http://localhost:8080/

Finally, terminate the running app, e.g. `CTRL+C` in the running application terminal window, and go back to the main directory, e.g. 

```shell
cd ..
```

## Deploy App
```shell
az spring app deploy -n ${APP_NAME} --artifact-path ./jars/demo-0.0.1-SNAPSHOT.jar
```

## Test App
Please go to the TEST endpoint to see the greeting message. Notice that TEST endpoint has automatically added authentication. 

## Add public endpoint
```shell
az spring app update -n ${APP_NAME} --assign-endpoint true
```

# Clean up
Let's clean up the application, Azure Spring Apps instance, and corresponding Resource Group.

## Delete App
```shell
az spring app delete -n ${APP_NAME}
```

## Delete Spring Apps Instance
```shell
az spring delete -n ${SPRING_APPS_SERVICE}
```

## Delete Resource Group
```shell
az group delete --name ${RESOURCE_GROUP}
```

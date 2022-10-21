# asae-demo

Instructions needed to run a simple asa-e hello world application. 

These instructions are a subset of the instructions at https://github.com/Azure-Samples/acme-fitness-store

# 1. Preparing the Azure Spring Apps

## 1.1 Pre-requisites

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

## 1.2 Install the Azure CLI extension

Install the Azure Spring Apps extension for the Azure CLI using the following command

```shell
az extension add --name spring
```

## 1.3 Prepare your environment for deployments

```shell
cp ./setup-env-variables-template.sh ./setup-env-variables.sh
```

Open `./setup-env-variables.sh` and update the following information:

```shell
export SUBSCRIPTION=subscription-id                 # replace it with your subscription-id
export RESOURCE_GROUP=resource-group-name           # existing resource group or one that will be created in next steps
export SPRING_APPS_SERVICE=azure-spring-apps-name   # name of the service that will be created in the next steps
export REGION=region-name                           # choose a region with Enterprise tier support
export APP_NAME=hello-world
```

Then, set the environment:

```shell
source ./setup-env-variables.sh
```

## 1.4 Login to Azure

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

## 1.5 Create Azure Spring Apps service instance

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

# 2. Deploying an application to Spring Azure Apps

## 2.1 Test hello-world app locally
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

## 2.2 Create App
```shell
az spring app create -n ${APP_NAME}
```

## 2.3 Deploy App
```shell
az spring app deploy -n ${APP_NAME} --artifact-path ./jars/demo-0.0.1-SNAPSHOT.jar
```

Alternatively, you can deploy the app directly from the source code, e.g.

```shell
az spring app deploy -n ${APP_NAME} --source-path hello-world
```

Also, you could pass in buildpack options through the command line, e.g. `BP_JVM_VERSION`

```shell
az spring app deploy -n ${APP_NAME} --source-path hello-world --env "BP_JVM_VERSION=17"
```

> Note: Observe the application deployment in Azure Portal.

If not provided `az spring app deploy` uses `default` as a default name for the deployment.
You could have multiple deployments of the same application (e.g. different versions).
A deployment can be set to `staging` or `production`.
Only one deployment can be set to `production` at the time. 

## 2.4 Test App

Observe the TEST point that was automatically generated upon deployment of the application.

> Note: Testing endpoint TEST endpoint is automatically generated, e.g.
https://primary:automaticallygeneratedpassword@xxxx.test.azuremicroservices.io/hello-world/default/
Observe that the HTTP username is always `primary`.
Observe that HTTP password is a very long sequence of alphanumeric characters.
Observe the url contains the name of the Azure Spring Apps service instance.
Observe the path contains the application name and the deployment name, which is `default` if no
deployment name was used.


## 2.5 Add public endpoint
```shell
az spring app update -n ${APP_NAME} --assign-endpoint true
```

>Note: Observe the new URL assigned to the application. Observe the `default` deployment and the status.

You can also get the URL programmatically, e.g.
```shell
az spring app show -n ${APP_NAME} --query "properties.url"
```

Also, could assign it to a variable, e.g. 
```shell
export APP_URL=$(az spring app show -n ${APP_NAME} | jq -r '.properties.url')
open ${APP_URL}
```

## 2.6 Observe the Actuator
Observe the Actuator endpoint for environment variables. Notice interesting environment variables
that are automatically added by the Azure Spring Apps (i.e. Kubernetes under the covers),
e.g. `/actuator/env/HOSTNAME`

```shell
export APP_URL=$(az spring app show -n ${APP_NAME} | jq -r '.properties.url')
curl -s ${APP_URL}/actuator/env/HOSTNAME
```

## 2.7 Scale out
Let's scale out the application by manually increasing the number of instances, e.g. 

```shell
az spring app scale -n ${APP_NAME} --instance-count 3
```

Observe the Actuator endpoint and see how the `HOSTNAME` changes with each request, e.g.
```shell
export APP_URL=$(az spring app show -n ${APP_NAME} | jq -r '.properties.url')
watch -n 2 curl -s ${APP_URL}/actuator/env/HOSTNAME
```

# 3. Clean up
Let's clean up the application, Azure Spring Apps instance, and corresponding Resource Group.

## 3.1 Delete App
```shell
az spring app delete -n ${APP_NAME}
```

## 3.2 Delete Spring Apps Instance
```shell
az spring delete -n ${SPRING_APPS_SERVICE}
```

## 3.3 Delete Resource Group
```shell
az group delete --name ${RESOURCE_GROUP}
```

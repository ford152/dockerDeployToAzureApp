# Dockerizing a node.js app and deploying an Azure App Service using the docker container

## Dockerizing
https://nodejs.org/en/docs/guides/nodejs-docker-webapp/#create-the-node-js-app

```
docker build . -t bb-testexpressapp
docker run --env PORT=3001 -p 2999:3001 -d bb-testexpressapp 
```

### Run Locally to Test
to call locally http://localhost:2999, which directs to the internal docker port 3001
to run in Azure:
```
docker run --env PORT=3001 -p 8080:3001 -d bb-testexpressapp 
```

## Create an Azure Container Registry and Push the Docker Container
https://docs.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-prepare-acr

I created the container registry in the portal, within the YOUNIQUE resource group
```
az login
az acr login --name YOUNIQUERegistry1
```

### Tag the docker image and push to the Azure container registry
First get the name of your private container registry
```
az acr show --name YOUNIQUERegistry1 --query loginServer --output table
```
Tag your docker container and push it
```
docker tag bb-testexpressapp youniqueregistry1.azurecr.io/bb-testexpressapp:v1

docker push youniqueregistry1.azurecr.io/bb-testexpressapp:v1
```

## List images in Azure container registry
Note that you can also go view in portal under "Repositories" in the container resource)
```
az acr repository list --name YOUNIQUERegistry1 --output table
or
docker images |egrep youniqueregistry1.azurecr.io
```

## Configure the Port for Azure to Access the Docker Container
By default, App Service assumes your custom container is listening on either port 80 or port 8080. If your container listens to a different port, set the WEBSITES_PORT app setting in your App Service app.  In the portal go to configuration, Application settings and add the new setting.

## Enable logging for the Azure app service that had the docker image deployed to it
```
az webapp log config --name bb-testexpressapp-docker1 --resource-group TEST --docker-container-logging filesystem

az webapp log tail --name bb-testexpressapp-docker1 --resource-group TEST
```

## Updating the container
Make your updates to your code and do a "docker build" to update it locally.  Push the updated docker container and specify a new tag when doing so:
```
docker tag bb-testexpressapp youniqueregistry1.azurecr.io/bb-testexpressapp:v2

docker push youniqueregistry1.azurecr.io/bb-testexpressapp:v2
```

In the portal go to Deployment Center and choose the new tag, which tells Azure to use the new verison of the container.



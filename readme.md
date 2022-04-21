# Dockerizing a node.js app and deploying an Azure App Service using the docker container

## Dockerizing
Create a Dockerfile and then build the docker container, giving it a name.  Details on the Dockerfile file in this article: https://nodejs.org/en/docs/guides/nodejs-docker-webapp/#create-the-node-js-app
```
docker build . -t expressapp1
```

### Run Locally to Test
```
docker run --env PORT=3001 -p 3000:3001 -d expressapp1 
```
Go to http://localhost:3000 in a browser.  Note that Docker is redirecting port 3000 to the internal port 3001.

View existing docker containers and stop the container that we just started and tested.
```
docker ps
docker stop 922d4008abfd  (take the CONTAINER ID value from the docker ps command output)
```

## Create an Azure Container Registry and Push the Docker Container
https://docs.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-prepare-acr

I created a container registry in the portal called "doItRegistry1".  Enable the "Admin user" under Access Keys for the container in the portal.  Log into Azure via Azure CLI and then log into the registry from the CLI:
```
az login
az acr login --name doItRegistry1
```

### Tag the docker image and push to the Azure container registry
Get the identifier of your private container registry:
```
az acr show --name doItRegistry1 --query loginServer --output table
```

Tag your docker container and push it. You'll need to reference the cotainer identifier from the output in the previous command.
```
docker tag expressapp1 doitregistry1.azurecr.io/expressapp1:v1

docker push doitregistry1.azurecr.io/expressapp1:v1
```

## List images in Azure container registry
Note that you can also go view in portal under "Repositories" in the container resource)
```
az acr repository list --name doItRegistry1 --output table
or
docker images |egrep doitregistry1.azurecr.io
```

## Create the Azure App Service
Using the portal, create an app service:
* Choose Docker Container as the Publish option
* On the Docker step, choose Azure Container Registry for the Image Source
* Choose the image called "expressapp1" and the tag "v1"

<br><br>

## Configure the Port for Azure to Access the Docker Container
By default, App Service assumes your custom container is listening on either port 80 or port 8080. If your container listens to a different port, set the WEBSITES_PORT app setting in your App Service app.  In the portal go to configuration, Application settings and add the new setting.  In this case the code in ./bin/www defaults to port 3000 (the PORT environment variable will not be passed to the docker image by default).  So I have set WEBSITES_PORT=3000.  Under the covers this causes Azure to launch docker using a command like so: docker run -d -p 3000:3000 .....
<br><br>

## Test the Docker Image That is Now Running as an App Service
https://expressapp-appservice1.azurewebsites.net
<br><br>
## Enable logging for the Azure app service that had the docker image deployed to it
Specify the App Service name as the --name parameter.

```
az webapp log config --name expressapp-appservice1 --resource-group TEST --docker-container-logging filesystem

az webapp log tail --name expressapp-appservice1 --resource-group TEST
```
You can also view the logs in the portal, App Service, Deployment Center, Logs.

## Updating the container
Make your updates to your code and do a "docker build" to update it locally.  Push the updated docker container and specify a new tag when doing so:
```
docker tag expressapp1 doitregistry1.azurecr.io/expressapp1:v2

docker push doitregistry1.azurecr.io/expressapp1:v2
```

In the portal go to Deployment Center and choose the new tag, which tells Azure to use the new verison of the container.

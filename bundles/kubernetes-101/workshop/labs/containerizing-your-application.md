# Containerizing your application

In this lab you will build your application, run it locally, and then package it into a container. Containers are more lightweight than a full virtual machine for your app and they package your application so that it runs the same across different environments such as development, QA, and production.

## Get the application code

The first hurdle to is the application itself.  How do you write it?  How do you deploy it?

## Build the app

Set the GOPATH variable, so that we can build our application.		
```bash
 export GOPATH=~/go
```

Build the app as a static binary.
```bash
cd ~/go/src/github.com/GoogleCloudPlatform/kubernetes-workshops/bundles/kubernetes-101/workshop
cd app/monolith
# main.go contains the app's entry point
go build -tags netgo -ldflags "-extldflags '-lm -lstdc++ -static'" .
```

## Test the app's functionality.
```bash
# Start up the application in the background.  Feel free to use different ports, if necessary.
./monolith --http :10180 --health :10181 &

# Test out the app's default functionality.
curl http://127.0.0.1:10180

# Attempt to access the app's secure endpoint.
curl http://127.0.0.1:10180/secure

# Get a JWT token from our application.  We'll use this to access the app's secure endpoint.
# The password for the next step is 'password'.
TOKEN=$(curl http://127.0.0.1:10180/login -u user | jq -r '.token')

# Examine the token, if you'd like.
echo $TOKEN

# Pass the token along the secure endpoint to get the message.
curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10180/secure
```

## Package the app
Once we have a working binary, we can use Docker to package it. 
```bash
# Check out the Dockerfile.  What is it doing?
cat Dockerfile

# Use the Dockerfile to build a new monolith image.
docker build -t <gcr.io>/<project_id>/monolith:1.0.0 .
```

After building the image, use Docker to verfiy that it still functions the same.
```bash
# Start an instance of your newly created docker image.
docker run -d <gcr.io>/<project_id>/monolith:1.0.0

# Use docker ps to get the container's ID
docker ps

# Use docker inspect to get find out the IP Address of your running container image.
docker inspect <container-id>

# Use the IP Address to test the instance's functionality.
curl http://<docker-ip>
```

## Choosing a container registry location

Your private registry name is your Google Cloud Platform project ID combined with a gcr.io hostname:

 - us.gcr.io hosts your images in the United States
 - eu.gcr.io hosts your images in the European Union
 - asia.gcr.io hosts your images in Asia
 - gcr.io without a prefix hosts your images in the United States, but this behavior may change in a future release


## Container Registry name format
The registry name format is below:

```bash
[HOSTNAME]/[YOUR-PROJECT-ID]/[IMAGE]
```

where

 - [HOSTNAME] is the gcr.io hostname
 - [YOUR-PROJECT-ID] is your Google Cloud Platform Console project ID
 - [IMAGE] is your image's name
 - [IMAGE_ID] is your image's ID (from docker images list)

For example, example-image from awesome-project, served in the United States, has the following registry name:
```bash
us.gcr.io/awesome-project/example-image
```

## Push the Image to Container Registry
To push to the registry using gcloud and docker, use '--' to pass docker args
```bash
# Upload the image into your container registry
gcloud docker -- push <gcr.io>/<project_id>/monolith:1.0.0
```

## Clean up
After verifying everything works as expected, clean up your environment.
```bash
# Stop the running docker container.
docker stop <container-id>

# Remove the docker container from the system.
docker rm <container-id>

# Remove the docker image from the system.
docker rmi <gcr.io>/<project_id>/monolith:1.0.0
```


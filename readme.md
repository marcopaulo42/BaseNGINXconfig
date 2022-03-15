# Create Base Docker NGINX Plus deployment

## Docker Deployment Instructions 

This deployment is based on the instructions detailed at https://www.nginx.com/blog/deploying-nginx-nginx-plus-docker/
specifically the section *"Deploying NGINX Plus with Docker"*.

These commands are run on the local host, assuming here Ubuntu.

## Step 1: Create Dockerfile

Create `Dockerfile` similar to **[NGINX Plus Dockerfile (Debian 11)](./Dockerfile)**

Which  exposes port 80 on the Container, and uses the default NGINX `nginx.conf` file.


## Step 2: Build NGINX Plus Image

With the `Dockerfile`, `nginx-repo.crt`, and `nginx-repo.key` files in the same directory, run the following command there to create a Docker image called `nginxplus` (as before, note the final period):

```# DOCKER_BUILDKIT=1 docker build --no-cache -t nginxplus --secret id=nginx-crt,src=</path/to/your/nginx-repo.crt> --secret id=nginx-key,src=</path/to/your/nginx-repo.key> .```

for example:

``# sudo DOCKER_BUILDKIT=1 docker build --no-cache -t nginxplus --secret id=nginx-crt,src=nginx-repo.crt --secret id=nginx-key,src=nginx-repo.key .``


The `DOCKER_BUILDKIT=1` flag indicates that we are using Docker BuildKit to build the image, as required when including the `--secret` option.

The `--no-cache` option tells Docker to build the image from scratch and ensures the installation of the latest version of NGINX Plus. If the `Dockerfile` was previously used to build an image and you do not include the `--no-cache option`, the new image uses the version of NGINX Plus from the Docker cache. (As noted, we purposely do not specify an NGINX Plus version in the Dockerfile so that the file does not need to change at every new release of NGINX Plus.) Omit the `--no-cache` option if it’s acceptable to use the NGINX Plus version from the previously built image.

The `--secret` option passes the certificate and key for your NGINX Plus license to the Docker build context without risking exposing the data or having the data persist between Docker build layers. The values of the id arguments cannot be changed without altering the base `Dockerfile`, but you need to set the src arguments to the path to your NGINX Plus certificate and key files (the same directory where you are building the Docker image if you followed the previous instructions).

## Sep 3: Verify Creation of Docker Image

Run command `# sudo docker images` to ensure that the image was created successfuly.

## Step 4: Create Docker Container

### Docker Container on port 80
Using the following commands to create the Docker Container:

`# sudo docker run --name mynginxplus -p 80:80 -d nginxplus`

### Docker Container on port 80 with directory mapping

The following command creates and runs the Container and maps the localhost directory for example: `/home/ubuntu/dockerbuildstuff/conf` to the Container directory `/etc/ninx/conf.d`.

`# sudo docker run -v /home/ubuntu/dockerbuildstuff/conf:/etc/nginx/conf.d --name mynginxplus -dp 80:80 nginxplus`

This is used to update the config files in the container through the mapping instead of directly in the Container.

## Extra

## To log into the running Docker Container 

Run command: 
`# sudo docker exec -it mynginxplus /bin/bash`





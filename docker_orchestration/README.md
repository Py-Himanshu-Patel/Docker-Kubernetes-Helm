# Docker Orchestration and Hosting a Platform

## Running applications with Docker Compose
Docker Compose (http:/​/​docs.​docker.​com/​compose/​) is the native Docker tool that runs interdependent containers that make up an application. We define a multicontainer application in a single file and feed it to Docker Compose, which sets up the application.
```bash
$ sudo pip install docker-compose
```
Write a docker compose file.
```yml
version: '3.1'
services:
  wordpress:
    depends_on:
      - mysql
    image: wordpress
    restart: always
    ports:
      - "8008:80"
    environment:
      WORDPRESS_DB_PASSWORD: example
  mysql:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example
```
Within the app directory, run the following command to build and start the app:
```bash
$ docker-compose up
```
Once the build is complete, access the WordPress installation page from `http://localhost:8080 or http://<host-ip>:8080`.
We can even build images from a Dockerfile during the compose, and then use it for the app.
```dockerfile
$ cat Dockerfile
FROM wordpress:latest
# extend the base wordpress image with anything custom
# you might need for local dev.
ENV CUSTOM_ENV env-value
```
We then need to update the `docker-compose.yml` file so that we are referencing the `Dockerfile` from above.
```yml
version: "3.1"
services:
  wordpress:
    build: .
    restart: always
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_PASSWORD: example
  mysql:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example
```
Once you have those changes, you start the stack the same way as before:
```bash
$ docker-compose up
```
To bring down the stack after you have started it, you do the following:
```
$ docker-compose down
```
Build the containers in the stack:
```
$ docker-compose build
```
Enter the running WordPress container:
```
$ docker-compose exec wordpress bash
```
List the running containers in the stack:
```
$ docker-compose ps
```

## Setting up a cluster with Docker Swarm
Docker Swarm mode supports two types of nodes; a **manager** and a **worker**. Manager nodes perform the orchestration and cluster management functions for the Swarm. They dispatch units of work called **tasks** to the **workers**. The manager nodes use the **Raft Consensus algorithm** (http:/​/​thesecretlivesofdata.​com/​raft/​) to manage the global cluster state. The worker nodes receive and execute the tasks that are dispatched by the managers.

For Raft to work correctly, you need an odd number of managers in order for the leader election to work properly. That means that, if you want a fault-tolerant Swarm cluster, you should have either three or five managers. If you have three managers, you can handle one manager node failure, and with five, you can handle up to two node failures before the Raft consensus is lost. Pick the cluster size that is most appropriate for your work load.

Refer Book for implementation

## Using secrets with Docker Swarm
When you are using containers, a common thing that you will need to do is connect to some external resources such as a database, cache, or web service. These resources usually need credentials. One popular way of passing these credentials to containers is to make them environmental variables that get populated on container startup. This allows you to use the same Docker image for different development environments, and removes the need for storing passwords in the image. This is a common trait for a twelve-factor app (https:/​/12factor.​net), which was made popular by Heroku (https:/​/​www.​heroku.​com).

Adding environmental variables to a container at runtime is very easy, but it has its downsides. When an environment variable is added to the container, it is accessible to everything running in that container. That means that, not only can your code see it, code from third-party libraries can as well. This makes it easy for those passwords to accidentally get exposed outside the container.

This usually happens accidentally when an error occurs, and in the resulting error stacktrace, it lists all of the current environment variables. This was originally added to help you debug the issue, but little did they know that, by exposing all of the environment variables, they were sharing your passwords as well, and creating another issue in the process. To fix this problem, Docker came out with a new feature called Docker Secrets (https:/​/docs.​docker.​com/​engine/​swarm/​secrets/​). Docker Secrets is currently only available to Swarm services. A secret can be any blob of data, such as a password, TLS certificate, and so on, that you don't want to share with others.

1. Add a secret to Swarm:
```
$ echo "myP@ssWord" | docker secret create my_password -
```
2. Create a service that uses the secret:
```
$ docker service create --name="my-service" --secret="my_password" redis
```
3. Since we are using Swarm, the container could be running on any of the nodes in the cluster. To find out which node your Swarm container is running on, you can use the `docker service ps` command:
```
$ docker service ps my-service
```
4. Now that you know which node the container is running on, connect to that node and run the following command to view the unencrypted secret inside the container:
```
$ docker container exec $(docker container ls --filter name=my-service -q) cat /run/secrets/my_password
```
It works like this, when you add a secret to Swarm, it will store this encrypted secret in its internal Raft store. When you create your service and reference that secret, Swarm will give those containers access to the secret, and it will add the unencrypted secret as an in-memory filesystem mount inside the container. In order to read the secret, your application will need to look at the filesystem mount instead of the environment.

If the secret is removed, or the service is updated to remove the secret, the secret will no
longer be available to the container.

Use the following code to inspect a secret:
```
$ docker secret inspect <secret name>
```
Use the following code to list the secrets:
```
$ docker secret ls
```
Use the following code to remove a secret:
```
$ docker secret rm <secret name>
```
Use the following code to update a service to remove the secret:
```
$ docker service update --secret-rm <secret name> <service name>
```

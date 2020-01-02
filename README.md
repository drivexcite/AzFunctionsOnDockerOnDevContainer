# Azure Functions running on a Docker Container running on a VS Remote Container

To add support for (seemingly) running a Docker container inside a VS Code Remote Container:

1.   Open the folder locally in VS Code.
2.   Add the following code in the `.devcontainer/Dockerfile`:
```bash
    apt-get update \
    && apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common lsb-release \
    && curl -fsSL https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]')/gpg | apt-key add - 2>/dev/null \
    && add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/$(lsb_release -is | tr '[:upper:]' '[:lower:]') $(lsb_release -cs) stable" \
    && apt-get update \
    && apt-get install -y docker-ce-cli; \
```
3.  Add the following code in the `.devcontainer/devcontainer.json`:
```json
"mounts": [
    "source=/var/run/docker.sock,target=/var/run/docker.sock,type=bind"
    ],
```
4.  Open a Terminal inside VS Code Remote Container.
5.  Build the container image by executing the following command:
```bash
docker build -t health-sample:v1 .
```
6.  Run the Container by executing the following command:
```bash
docker run --publish 0.0.0.0:8089:80 health-sample:v1 -rm
```
7. On the Docker host, navigate to [http://localhost:8099/api/Health](http://localhost:8099/api/Health).

## How it works
* On Step #2, the script adds the Docker CLI, but doesn't actually install the daemon. This steps allows issuing docker commands inside the VS Remote Container.
* On Step #3, it binds the Docker host where the VS Remote Container is running to the Docker CLI running inside it. This way, we can build an image and run a container and it will be directly accessible via Docker port forwarding in the Docker Host loopback interface, without any need for the `appPort` setting inside the `.devcontainer/devcontainer.json` file.

However, in the case of the a Function App project, it is still a good idea to use the remote container port forwarding plus the Azure Function emulator to easily debug C# code.

## References
-   [Docker in Docker](https://github.com/Microsoft/vscode-dev-containers/tree/master/containers/docker-in-docker)
-   [Using Docker or Kubernetes inside a Container](https://code.visualstudio.com/docs/remote/containers-advanced#_using-docker-or-kubernetes-from-a-container)

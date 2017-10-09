# Codefresh | Docs

# Use Codefresh to Build and Deploy Containers to Microsoft Azure | ACS Swarm (Pre 1.12)

## Before we begin

This document will walk you through deploying to Azure Container Service using Swarm (pre version 1.12) as your Orchestrator, which does not support Docker stack at this time.  

> Note: If you are looking for documentation on how to deploy to a [Docker Swarm Mode Cluster on Microsoft Azure using Azure Container Engine][1] or a [DockerCE cluster on Azure Container Service][2], **please see this [document][3] instead**. To learn more about the differences in Azure Container Service Docker Swarm and Docker Swarm Mode with ACS Engine, check out [Microsoft's FAQ][4].

## Getting started

Download [Docker][5]. If you are on Mac or Windows, [Docker Compose][6] will be automatically installed. On Linux, make sure you have the latest version of [Compose][7].

### Try this example locally 

1. First, open an SSH Tunnel to your Swarm Master and export an environment variable for DOCKER_HOST.
  - `ssh -fNL 2375:localhost:2375 -p 2200 azureuser@azure.westcentralus.cloudapp.azure.com`
  - `export DOCKER_HOST=:2375`
2. Second, make sure you have a swarm. If you don't, run:
  - `docker swarm init`

> Note: If you used Azure Container Service using Swarm as your Orchestrator, your swarm is already initialized.

Once you have your swarm, in this directory run:

> docker run -d --name Demo -p 80:80 nginx

## Deploy to Remote Swarm with codefresh.yml    
    
    deploy_to_swarm:
      image: ncodefresh/remote-docker:azure
      working_directory: ${{main_clone}}
      commands:
        -rdocker ${{RDOCKER_HOST}} ${{LOCAL_PORT}} docker run -d --name ${{CONTAINER_NAME}} -p 80:80 ${{IMAGE_NAME}}
      environment:
        - SSH_KEY=${{SSH_KEY}}
        - SSH_PORT=${{SSH_PORT}}

> RDOCKER_HOST remote Docker swarm master machine, accessible over SSH (for example, azureuser@azure.southcentralus.cloudapp.azure.com)

> LOCAL_PORT is port 2375 - the port your Docker Swarm Cluster is listening on

> CONTAINER_NAME is new Docker container name (use "demo", for example)

> IMAGE_NAME is the Docker image name (use "nginx", for example)

> SSH_KEY private SSH key, used to access Docker swarm master machine

> SPLIT_CHAR split character, you've used to replace `newline` in SSH key. Recommendation: use `,` (`comma` character)

> SSH_PORT is port 2200 (default SSH Port for ACS Engine and Azure Container Service with Swarm / DockerCE orchestrator)

> Note: Ports 80, 443, and 8080 are open by default on your Agent Load Balancer in Azure when using ACS Swarm, ACS Engine, or ACS with DockerCE (Preview)

### Passing SSH key through ENV variable 

Currently in order to pass SSH key through Codefresh UI, you need to convert it to single line string (replacing `newline` with `comma`), like this:
    
    
    $ SSH_KEY=$(cat ~/.ssh/my_ssh_key_file | tr 'n' ',')
    

1. Select **Codefresh's Deploy Images** in the pipeline's and select `ncodefresh/remote-docker:azure`. Be sure to use the ssh-tunnel branch.

> Note: You can also add `ncodefresh/remote-docker:azure` to your Codefresh.yml file as provided in the example above.

2. As a deploy command use `rdocker ${{RDOCKER_HOST}} docker stack deploy --compose-file docker-stack.yml ${{STACK_NAME}}` .

3. Make sure you define the following variables in the pipeline as defined in the previous part:

    * `STACK_NAME`
    * `RDOCKER_HOST`
    * `SSH_KEY`
    * `SPLIT_CHAR`
    * `SSH_PORT`

![][6]

**Notice:** The UI deploy step will run on any build. Make sure that your automated builds run only on a specific branch trigger.

[1]: https://github.com/Azure/azure-quickstart-templates/tree/master/101-acsengine-swarmmode
[2]: https://docs.microsoft.com/en-us/azure/container-service/dcos-swarm/container-service-swarm-mode-walkthrough
[3]: Swarm-mode.md
[4]: https://docs.microsoft.com/en-us/azure/container-service/kubernetes/container-service-faq
[5]: https://www.docker.com/products/overview
[6]: https://docs.docker.com/compose
[7]: https://docs.docker.com/compose/install/

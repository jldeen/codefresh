# Codefresh | Docs

# Deploy Codefresh to Microsoft Azure 

## Before we begin

This document will walk you through deploying to a [Docker Swarm Mode Cluster on Microsoft Azure using Azure Container Engine][10]. 

The method outlined in this document will also work for deployment to a [DockerCE cluster on Azure Container Service][8]. However, DockerCE clusters on ACS are currently in preview and are not recommended for production environments. 

> Note: If you are looking for documentation on how to deploy to Azure Container Service using Swarm as your Orchestrator, **please see this [document][7] instead**. To learn more about the differences in Azure Container Service Docker Swarm and Docker Swarm Mode with ACS Engine, check out [Microsoft's FAQ][9].

## Getting started

Download [Docker][1]. If you are on Mac or Windows, [Docker Compose][2] will be automatically installed. On Linux, make sure you have the latest version of [Compose][3].

### Try this example 

> Just head over to the example [**repository**][4] in Github.

To run the provided example on a [Docker Swarm][5], first make sure you have a swarm. If you don't, run:

> docker swarm init

> Note: If you used the [101 ACS Swarm Mode GitHub QuickStart Template][10] or a [DockerCE Cluster (preview) on Azure Container Service][8], your swarm is already initialized.

Once you have your swarm, in this directory run:

> docker stack deploy --compose-file docker-stack.yml vote
    
    deploy_to_swarm:
      image: ncodefresh/remote-docker:ssh-tunnel
      working_directory: ${{main_clone}}
      commands:
        - rdocker ${{RDOCKER_HOST}} docker stack deploy --compose-file docker-stack.yml ${{STACK_NAME}}
      environment:
        - SSH_KEY=${{SSH_KEY}}
        - SSH_PORT=${{SSH_PORT}}
      when:
        branch:
          only:
            - ssh-tunnel

> RDOCKER_HOST remote Docker swarm master machine, accessible over SSH (for example, azureuser@azure.southcentralus.cloudapp.azure.com)

> STACK_NAME is new Docker stack name (use "vote", for example)

> SSH_KEY private SSH key, used to access Docker swarm master machine

> SPLIT_CHAR split character, you've used to replace `newline` in SSH key. Recommendation: use `,` (`comma` character).

> SSH_PORT is port 2200 (default SSH Port for ACS Engine and Azure Container Service with Swarm / DockerCE orchestrator)

The app will be running at the FQDN of the swarm agent pool on port 80 and the results will be accessible at port 8080. Examples:
> Vote App: http://azure-agent.southcentralus.cloudapp.azure.com
> Results: http://azure-agent.southcentralus.cloudapp.azure.com:8080

> Note: Ports 80, 443, and 8080 are open by default on your Agent Load Balancer in Azure when using ACS, ACS Engine, or ACS with DockerCE (Preview)

### Passing SSH key through ENV variable 

Currently in order to pass SSH key through Codefresh UI, you need to convert it to single line string (replacing `newline` with `comma`), like this:
    
    
    $ SSH_KEY=$(cat ~/.ssh/my_ssh_key_file | tr 'n' ',')
    

1. Select **Codefresh's Deploy Images** in the pipeline's and select `ncodefresh/remote-docker:azure`. Be sure to use the ssh-tunnel branch.

2. As a deploy command use `rdocker ${{RDOCKER_HOST}} docker stack deploy --compose-file docker-stack.yml ${{STACK_NAME}}` .

3. Make sure you define the following variables in the pipeline as defined in the previous part:

    * `STACK_NAME`
    * `RDOCKER_HOST`
    * `SSH_KEY`
    * `SPLIT_CHAR`
    * `SSH_PORT`

![][6]

**Notice:** The UI deploy step will run on any build. Make sure that your automated builds run only on a specific branch trigger.

[1]: https://www.docker.com/products/overview
[2]: https://docs.docker.com/compose
[3]: https://docs.docker.com/compose/install/
[4]: https://github.com/codefreshdemo/example-voting-app
[5]: https://docs.docker.com/engine/swarm/
[6]: https://files.readme.io/0a66a41-image3.png
[7]: https://github.com/codefreshdemo/example-voting-app
[8]: https://docs.microsoft.com/en-us/azure/container-service/dcos-swarm/container-service-swarm-mode-walkthrough
[9]: https://docs.microsoft.com/en-us/azure/container-service/kubernetes/container-service-faq
[10]: https://github.com/Azure/azure-quickstart-templates/tree/master/101-acsengine-swarmmode
  
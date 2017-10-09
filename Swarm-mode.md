# Codefresh | Docs

# Use Codefresh to Build and Deploy Containers to Microsoft Azure | ACS Engine or DockerCE via ACS

## Before we begin

This document will walk you through deploying to a [Docker Swarm Mode Cluster on Microsoft Azure using Azure Container Engine][1]. 

The method outlined in this document will also work for deployment to a [DockerCE cluster on Azure Container Service][2]. However, DockerCE clusters on ACS are currently in preview and are not recommended for production environments.

Part of this quickstart requires Azure CLI version 2.0.4 or later. From command line, run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI 2.0][3]

> Note: If you are looking for documentation on how to deploy to Azure Container Service using Swarm as your Orchestrator, **please see this [document][4] instead**. To learn more about the differences in Azure Container Service Docker Swarm and Docker Swarm Mode with ACS Engine, check out [Microsoft's FAQ][5].

## Getting started

Download [Docker][6]. If you are on Mac or Windows, [Docker Compose][7] will be automatically installed. On Linux, make sure you have the latest version of [Compose][8].

### Try this example locally 

> Just head over to the example [**repository**][9] in Github.

To run the provided example on a [Docker Swarm][10] from your local command line, 

1. First, open an SSH Tunnel to your Swarm Master and export an environment variable for DOCKER_HOST.
  - `ssh -fNL 2375:localhost:2375 -p 2200 azureuser@azure.westcentralus.cloudapp.azure.com`
  - `export DOCKER_HOST=:2375`
2. Second, make sure you have a swarm. If you don't, run:
  - `docker swarm init`

> Note: If you used the [101 ACS Swarm Mode GitHub QuickStart Template][1] or a [DockerCE Cluster (preview) on Azure Container Service][2], your swarm is already initialized.

Once you have your swarm, in your cloned directory run:

  - `docker stack deploy --compose-file docker-stack.yml vote`

## Deploy to Remote Swarm with codefresh.yml
    
    deploy_to_swarm:
      image: ncodefresh/remote-docker:azure
      working_directory: ${{main_clone}}
      commands:
        - rdocker ${{RDOCKER_HOST}} docker stack deploy --compose-file docker-stack.yml ${{STACK_NAME}}
      environment:
        - SSH_KEY=${{SSH_KEY}}
        - SSH_PORT=${{SSH_PORT}}

> RDOCKER_HOST remote Docker swarm master machine, accessible over SSH (for example, azureuser@azure.westcentralus.cloudapp.azure.com)

> STACK_NAME is new Docker stack name (use "vote", for example)

> SSH_KEY private SSH key, used to access Docker swarm master machine

> SPLIT_CHAR split character, you've used to replace `newline` in SSH key. Recommendation: use `,` (`comma` character).

> SSH_PORT is port 2200 (default SSH Port for ACS Engine and Azure Container Service with Swarm / DockerCE orchestrator)

> LOCAL_PORT is port 2375 (port Docker Swarm-mode cluster is listening on)

The app will be running at the FQDN of the swarm agent pool on port 80 and the results will be accessible at port 8080. Examples:
> Vote App: http://azure-agent.southcentralus.cloudapp.azure.com
> Results: http://azure-agent.southcentralus.cloudapp.azure.com:8080

> Note: Ports 80, 443, and 8080 are open by default on your Agent Load Balancer in Azure when using ACS Swarm, ACS Engine, or ACS with DockerCE (Preview). To quickly access the FQDN for both your Master and Agent load balancers, run the following command from your local command line:

```bash
az acs list --resource-group myResourceGroup --query '[*].{Master:masterProfile.fqdn,Agent:agentPoolProfiles[0].fqdn}' -o table
```
Output:
```
Master                                                               Agent
-------------------------------------------------------------------  --------------------------------------------------------------------
myswarmcluster-myresourcegroup-d5b9d4mgmt.westcentralus.cloudapp.azure.com  myswarmcluster-myresourcegroup-d5b9d4agent.westcentralus.cloudapp.azure.com
```
> Note: You can also use the `-g` switch in place of `--resource-group`

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
    * `LOCAL_PORT`


**Notice:** The UI deploy step will run on any build. Make sure that your automated builds run only on a specific branch trigger.

[1]: https://github.com/Azure/azure-quickstart-templates/tree/master/101-acsengine-swarmmode
[2]: https://docs.microsoft.com/en-us/azure/container-service/dcos-swarm/container-service-swarm-mode-walkthrough
[3]: https://docs.microsoft.com/cli/azure/install-azure-cli
[4]: Swarm.md
[5]: https://docs.microsoft.com/en-us/azure/container-service/kubernetes/container-service-faq
[6]: https://www.docker.com/products/overview
[7]: https://docs.docker.com/compose
[8]: https://docs.docker.com/compose/install/
[9]: https://github.com/codefreshdemo/example-voting-app
[10]: https://docs.docker.com/engine/swarm/

Use Codefresh to Build and Deploy Containers to Microsoft Azure
=========

Codefresh Docker CI/CD service is able to deply the voting application to specified remote swarm cluster on Microsoft Azure. To learn more, please use the following links.

---------------

Getting started
---------------

1. [Swarm Mode via ACS Engine and DockerCE (Preview)][1]
    - The method outlined in this document will work for deployment to a [Docker Swarm Mode Cluster on Microsoft Azure using Azure Container Engine][4] or a [DockerCE cluster on Azure Container Service][5]. 
    
    > Note: DockerCE clusters on ACS are currently in preview and are not recommended for production environments.   

2. [ACS for Docker Swarm (Pre 1.12)][2]
    - The method outlined in this document will work for deployment to a Docker Swarm Cluster (not swarm mode, pre Docker engine 1.12) on Microsoft Azurew using Azure Container Service.

3. [Example Voting App][3]
    - The code in this repo is simply a demo app you can use to deploy to Swarm Mode via ACS Engine or DockerCE (Preview)

[1]: Swarm-mode.md
[2]: Swarm.md
[3]: https://github.com/jldeen/example-voting-app/tree/ssh-tunnel
[4]: https://github.com/Azure/azure-quickstart-templates/tree/master/101-acsengine-swarmmode
[5]: https://docs.microsoft.com/en-us/azure/container-service/dcos-swarm/container-service-swarm-mode-walkthrough

# Deploy and Manage a Scalable Payara Cluster on Azure

This repo contains templates that can deploy a [Payara](https://payara.fish) cluster consisting of one admin server and a number of application servers.

# How to deploy

## Azure CLI (Azure Cloud Shell recommended)

You'll want to use Azure CLI to deploy the templates, and [Azure Cloud Shell](https://shell.azure.com/) might be best
(Azure CLI is already installed and configured), if you don't already have a machine installed and set up with Azure CLI.
Make sure that your subscription is set correctly by checking `az account show` resul (you can set your subscription by
`az account set -s <your_desired_subscription_id>`).

## Get the templates

The templates are hosted on Github. Check out the templates into your cloud shell directory:

```
git clone https://github.com/hosungsmsft/azure-quickstart-templates/
cd azure-quickstart-templates/payara-ubuntu-cluster
git checkout payara
```

## Create an SSH key pair

We need an SSH key pair to configure deployed VMs for remote SSH access. Create a key pair using the following command:

```
ssh-keygen -t rsa -N "" -f my_payara_id_rsa
```

Feel free to replace the file name (`my_payara_id_rsa`) to your desired name.

## Create a resource group

We should create an Azure resource group where we'll deploy the templates:

```
az group create -g my_payara_rg -n <desired_azure_region>
```

## Deploy templates

Use the following command to deploy the templates. If you used your own names for files and the resource group,
make sure to replace them in the command. You'll also need to provide a good password for the `payaraAdminServerPassword'
parameter (you'll use this parameter to access your deployed Payara admin server).

```
az group deployment create -g my_payara_rg --template-file azuredeploy.json --parameters sshPublicKey="$(cat my_payara_id_rsa.pub)" sshPrivateKey="$(cat my_payara_id_rsa | base64 -w 0)" payaraAdminServerPassword=<your_desired_payara_admin_password>
```

## Accessing the deployed resources (controller VM and Payara site)

Deployed controller VM (where Payara Admin Server runs) and the Payara site (application server cluster) can be accessed as follows.

First of all, you'll need to get the domain name of the controller VM and the load balancer:

```
ctlrDNS=$(az group deployment show -g my_payara_rg -n azuredeploy --query properties.outputs.controllerDNS.value)
lbDNS=$(az group deployment show -g my_payara_rg -n azuredeploy --query properties.outputs.loadBalancer.value)
```

An example controller DNS is like `controller-pubip-xyz123.someregion.cloudapp.azure.com`. An example load balancer DNS is
like `lb-xyz123.someregion.cloudapp.azure.com`.

You can login to the controller VM using SSH with the following command:

```
ssh -i my_payara_id_rsa payaraadmin@$ctlrDNS
```

The Payara admin server web console can be accessed by browsing `https://$ctlrDNS:4848`. You should use `admin` as the login ID and the `payaraAdminServerPassword` parameter value you used in the `az group deployment create` command above as the login password.

The deployed Payara application site can be accessed by browsing `https://$lbDNS` or `http://$lbDNS`.

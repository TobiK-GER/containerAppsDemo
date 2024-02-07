### cli commands to set up container apps environment (without workload profile) and container app

1) Get containerapp az cli extension

az extension add --name containerapp --upgrade

2) register provider if never used in subscription

az provider register --namespace Microsoft.App

3) create resource group, storage account, vnet with subnet (of at least /23 size)

az group create -n containerapp-rg -l westeurope

az storage account create -g containerapp-rg -n containerapptbkusa -l westeurope --sku Standard_LRS --public-network-access Enabled

az network vnet create -g containerapp-rg -n containerappenv-vnet -l westeurope --address-prefixes 10.4.0.0/20 --subnet-prefixes 10.4.0.0/23 --subnet-name subnetcontappenv

4) create container app environment with logging to storage account, using existing subnet, consumption only - no workload profiles

az containerapp env create -n tbkutestcontappenv -g containerapp-rg -l westeurope --logs-destination azure-monitor --storage-account containerapptbkusa --enable-workload-profiles false --platform-reserved-cidr 10.0.0.0/20 --platform-reserved-dns-ip 10.0.0.2 --infrastructure-subnet-resource-id "/subscriptions/89cfdc40-ac1f-443a-bbb1-05f9a341334d/resourceGroups/containerapp-rg/providers/Microsoft.Network/virtualNetworks/containerappenv-vnet/subnets/subnetcontappenv"

5) create container app - vanilla js

az containerapp create -n tbkutestapp -g containerapp-rg --ingress external --environment tbkutestcontappenv --cpu 0.25 --memory 0.5 --target-port 8080 --image nginx
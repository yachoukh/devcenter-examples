# Setup steps using az cli

1. [Create dev center resource](#create-dev-center-resource)
2. [Create dev box definition](#create-dev-box-defitition)
3. [Create project under dev center](#create-project)
4. [Create a host pool under project](#create-devbox-pool)

## Set up variables

```bash
devcenterName=YourDevCenterName
projectName=projectX
location=swedencentral
resourceGroupName=test-rg
```


## Create dev center resource

```bash
 devcenterId=$(az devcenter admin devcenter create --name $devcenterName --resource-group $resourceGroupName --location $location --query id -o tsv)
```

## Create dev box defitition

First get the id of the default image gallery

```bash
galleryId=$(az devcenter admin gallery list -d $devcenterName -g $resourceGroupName --query [].id -o tsv)
```
We will use the visual studio image with hibernation support.

```bash
imageId=$galleryId/images/microsoftvisualstudio_visualstudioplustools_vs-2022-ent-general-win11-m365-gen2

```

Now you'll create the dev box definition with 8 cores, 32gb and 256 GB OS disk plus hibernation enabled

```bash
az devcenter admin devbox-definition create -g $resourceGroupName -d $devcenterName -n 'def1' --image-reference 'id=$imageId' --os-storage-type 'ssd_256gb' --sku 'name=general_i_8c32gb256ssd_v2' --hibernate-support 'Enabled'
```

## Create project

```bash
az devcenter admin project create --location $location --dev-center-id $devcenterId --name $projectName --resource-group $resourceGroupName
```

## Create devbox pool

```bash
DEV_BOX_POOL_ID=$(az devcenter admin pool create --devbox-definition-name 'def1' --name 'pool1' -g $resourceGroupName --project $projectName --single-sign-on-status 'Enabled' --local-administrator 'Enabled' --stop-on-disconnect status="Enabled" grace-period-minutes="60" --virtual-network-type 'Managed' --managed-virtual-network-regions $location   --query id -o tsv)
```


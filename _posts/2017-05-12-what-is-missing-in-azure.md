# Azure related

TL;DR. It's all about Azure related.
Personally I complain Azure every day every time, But Azure improve a lot, not only because of lots of my formal colleagues are working with Azure, but also they really want to do things good, however not good enough as a product.
They did really great job with their product, and they towards to community.
But As a prodcut, they are far from AWS yet.

## Backgroud

Microsoft had release Azure for a long time, Here we only talk about Azure ARM.
We are using Azure cloud, not Azure suits which offering OneDrive or Excel online for both users and enterprise users.

We start to evulate for Azure service long time ago, when I joined the team, they almostfinished desgin.

## Infrastructure as code

### Template

[Infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_Code) was very close to DevOPs. especially, during the time, AWS using cloudformation.
As the similar approch, Azure is using [ARM template](https://docs.microsoft.com/en-us/azure/azure-resource-manager/), which defines depends and create order for each services. You can find most useful templates from [github repository](https://github.com/Azure/azure-quickstart-templates), With the open mindset from Microsoft, we can get more resources and document before, unfortunately, all the resources are seperating in each different area, the development team, document team and support team have there own channel to talk, although, their internal faces are effienacy enough. but still, it caused problem, at least for us.

The template has [schema](https://github.com/Azure/azure-resource-manager-schemas/tree/master/schemas), you won't get notified that when they are updated to their schema, or something changed there.

### CLI

Azure provide good tool for managing Azure services.

#### Powershell

Powershell belongs to Microsoft, according to their support engineer, they would love to request all the support users to try to run with powershell, in most of their document, powershell is the first choice.
As Powershell was available for both Linux and Mac, Microsoft want to expend their usage for *nix like system. unfortuantely, [powershell for azure](https://github.com/Azure/azure-powershell), is not able to running that well on Linux and Mac OS X with powershell, you have to install [nuget version](https://blogs.technet.microsoft.com/jessicadeen/azure/getting-started-with-powershell-core-and-azurerm-modules-on-ubuntu-and-os-x/) accordly, and this preview version is missing lots of the function. As one of the issue tracker, they will release complete full version the second half of the year.
However, Powershell is the most complete CLI for Azure, it provides most the function that the system admin need.

#### Azure CLI

##### Azure CLI 1.0

This [command](https://docs.microsoft.com/en-us/azure/cli-install-nodejs) is based on nodejs. As the notice mentioned.
>This topic describes how to install the Azure CLI 1.0, which is built on nodeJs and supports all classic deployment API calls as well as a large number of Resource Manager deployment activities. You should use the Azure CLI 2.0 for new or forward-looking CLI deployments and management.

Azure now is more like people to migrate to ARM portal. so let's talk about Azure CLI 2.0

##### Azure CLI 2.0

[Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) is based on python, used to be in preview, April, 2017, they officially remove the preview tag. You can raise issues in [github](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli). Azure CLI 2.0 is most fitable for *nix administrator.

**Pros:**

* It can be easy intergrited.
* Output is Json.

**Cons:**

* Still missing some function
* Document is not enough, if you want looking for the usage, it's better to search the source code.

##### Azure RESTful API

Actually, all the CLI is calling the [API](https://docs.microsoft.com/en-us/rest/api/).
So far,  the API document is clear enough, probably missing some example. When the API version changed, you might not able to get the latest version. Stick on document, that probably the safest way.

One thing I'd like to hightlight, in web apps, there is a function called swapwithpreview, but in the API, it's called "preserveVnet", that really confused. You have to read both command line source code and RESTful API document.

### External Tools

Terraform is a good tool as IaC tools, <https://www.terraform.io/docs/providers/azurerm/index.html>.
But the diffault part is creating service princple, and use service princple.

```bash
  az login
  az account set --subscription <Subscription ID>
  az ad sp create-for-rbac
```

Generally, they are having azure cli 1.0 [script](https://github.com/Azure/azure-devops-utils/blob/master/bash/create-service-principal.sh)

## Azure Virtual Machine

### Virtual Machine

You can use both command way and template way to create the VM without problem. and Also with your provision chef/puppet/ansible to create as well.
Here we are talking about linux and chef only. Azure provide a [extension](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/extensions-features). we will use custom script and chef. We won't talk about how to use it, because of the document already mentioned, here is what we found.

When you are running custom script, you have to exclude upgrade waagent

```bash
  yum update -y --exclude=WALinuxAgent
```

Otherwise walinuxagent will kill itself, and stuck at update forever.

When you are using template, I would suggest you set autoupgrade to false, because of during the upgrade, it might delete the your original running script "/var/lib/waagent/custom-script", but that depends on you!

```bash
  "autoUpgradeMinorVersion": false
```

Here is the [script](https://github.com/Azure/azure-quickstart-templates/blob/master/az-group-deploy.sh) to use with deploy with sastoken, for us we are using perl to replace SAS token in the parameter file, due to '&' will get problem to replace.

```bash
   perl -p -i -e
```

### NTP

Azure didn't provide NTP services for linux, you may use ntp service with *nix destro default one.

### Internal DNS

Azure image contains dnsmasq, it's very easy to setup with your own resolve servers. and it's allow you to point this dns server in the dns level.
P.S. VM is no problem at all, when we point our internal web apps, it never get the right dns settings.

### Resource group

Azure didn't mention that each resource will have their own internal domain. For instance, I got resource group foorg, and the other barrg. one vm x in foorg, y in barrg.

x has FQDN with x.(random with foorg).mx.internal.cloudapp.net
y has FQDN with y.(random with barrg).mx.internal.cloudapp.net

Although, they are in the same subscription, their vnet set to peering, still the are not able to resolve each other, unless they are pointing to the same internal dns or modify in the /etc/hosts.

## Azure Web Apps

In the other hand, when you are using pfx like certification, the template simply accept base64 encryption. you set the certification with base64 encryption, and replace "/" with "#", and replace "#" again, otherwise, it will have problem.

```bash

CERT_DATA=$(base64 $CERT_FILE |  tr -d '\n' |sed -e 's/\//#/g')
CERT_PASSWORD=$(cat $CERT_PASSWORD_FILE)

function param_replace {
    perl -p -i -e "s/CERT_DATA_GOES_HERE_REPLACE_ME/$CERT_DATA/g" $VM_PARAMS
    perl -p -i -e "s/#/\//g" $VM_PARAMS
    perl -p -i -e "s/CERT_PASSWORD_GOES_HERE_REPLACE_ME/$CERT_PASSWORD/g" $VM_PARAMS
}
```

### Azure Storage

All the logs shows that you can use sdk to upload the content/data to storage. Here is the template how we backup data, it's python, but mixed with ruby :(

```python
"""Backup jenkins job and history """
from datetime import date, timedelta
import os
import tarfile
from azure.storage.blob import BlockBlobService

AZURE_STORAGE_ACCOUNT = "<%= @storageaccount %>"
AZURE_STORAGE_ACCESS_KEY = \
   "<%= @storagekey %>"
CONTAINER = "<%= @container %>"
WORKDIR = '/var/lib/jenkins'

time = date.today()
CURRENT = time.strftime("%Y%m%d")

SERVICE = BlockBlobService(account_name=AZURE_STORAGE_ACCOUNT, account_key=AZURE_STORAGE_ACCESS_KEY)

os.chdir(WORKDIR)

def compress(folder_name):
    """get compress and upload"""
    tar_name = folder_name + '_' + CURRENT + '.tar.gz'
    blob_name = "jenkins/" + tar_name
    tar = tarfile.open(tar_name, "w")
    tar.add(folder_name)
    tar.close()
    SERVICE.create_blob_from_path(CONTAINER, blob_name, tar_name)
    os.remove(tar_name)


for folder in ['jobs', 'config-history']:
    compress(folder)
    print('Successfully Upload!')
```

Actually there is more easy way, just using curl. which is not published in Azure document.
It's not listed in any documents.

```bash
curl -X PUT -H "x-ms-blob-type: BlockBlob" -H "x-ms-blob-content-type: application/x-www-form-urlencoded" \
  "${STORAGE_URL_CONTAINER}${UPLOAD_NAME}${SAS_TOKEN}" --data-binary "@${PATH}"
```

### Get the bear code

If you don't like to command way, you can call for the API. when you are doing the post, you have to get the bear code as token. Azure provide a very nice [document](https://github.com/Microsoft/azure-docs/blob/master/articles/azure-resource-manager/resource-manager-rest-api.md)
In short words, better to use service princple to do this.

```bash
a=$(curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=client_credentials&resource=https://management.core.windows.net/&client_id=<application id>&client_secret=<password you selected for authentication>" "https://login.microsoftonline.com/<Azure AD Tenant ID>/oauth2/token?api-version=1.0" | jq ".access_token" > /dev/null)

curl -s -H "Authorization: Bearer ${a}" -X POST "https://management.azure.com/subscriptions/<subscription id?/resourceGroups/<resource group>/providers/Microsoft.Web/sites/<resource name>/resetSlotConfig?api-version=2016-08-01"
```

Then you will get the token

### Subnet

Azure is not able to devide subnet as AWS
for example, you have a range 10.0.0.0/22. you can create subnet with 10.0.1.0/24 and 10.0.2.0/24, you are not able to use 10.0.1.0/28 then 10.0.1.16/28

### To Be Contunied
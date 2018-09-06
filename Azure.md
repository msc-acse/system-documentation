# Microsoft Azure

Imperial members can log in to the Azure web interface at https://portal.azure.com.

In order to create new virtual machines, it requires you to be given the right permissions to use the correct subscription (i.e. to tell them who to bill).


## Command Line Tools

The Azure command line interface 'az' can be used to script interactions with Azure. Installation and usage instructions are available (here)[https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest].

Commands:

 - Log in: `az login`
 - Create a machine: `az vm create -g <resource group> -n <vm name> --image UbuntuLTS   --size Standard_B1s --admin-password <your password>  --custom-data cloud_init.yaml`
 - Start machine: `az vm start -g <resource group> -n <vm name>`
 - Stop machine: `az vm stop -g <resource group> -n <vm name>`
 - Restart machine: `az vm restart -g <resource group> -n <vm name>`
 - Delete machine: `az vm delete -g <resource group> -n <vm name>` 
 - Add AAD extensions: `az vm extension set --publisher Microsoft.Azure.ActiveDirectory.LinuxSSH --name AADLoginForLinux -g <resource group> --vm-name <vm name>`
 - Open an incoming port: `az vm open-port -g <resource group> -n <vm name> --port <port no.> --priority <integer number>`
 - Allow AD login (admin): `az role assignment create --role “Virtual Machine Administrator Login” --assignee <username>@ic.ac.uk \
   --scope $(az vm show -g <resource -n <VM name> --query id -o tsv)`
 - Allow AD login (user): `az role assignment create --Role “Virtual Machine User Login” --assignee <username>@ic.ac.uk \
   --scope $(az vm show -g <resource -n <VM name> --query id -o tsv)`
   

### Cloud-init

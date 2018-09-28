# Setting up JupyterHub on a local or cloud machine

See also  https://github.com/jupyterhub/jupyterhub-tutorial/blob/master/JupyterHub.pdf for a more general discussion.

## Installing Jupyterhub

For a generic Ubuntu installation you can use:

    sudo python3 -m pip install jupyterhub
    npm install -g configurable-http-proxy

Or for one with conda installed:

    conda install -c conda-forge jupyterhub
    conda install notebook

To test, you can start up a default configuration from your termninal

    sudo jupyterhub

This will start the jupyterhub server with default settings, using local authentication (only users who can log into that computer can connect) and local spawning (this assumes the ability to switch users, hence the `sudo` above). It serves a login page accessable at `http://[the computer ip]:8000`. If the browser is on the same system as the server, then you can just use `http://localhost:8000`.

## SSL certificates

To serve ssl encrypted, it's necessary to provide signed ssl certificates. The Azure VM installations come with self-signed certificates (which raises a warning on users web-browsers). It is also possible to replace these with CA signed ones if availabe. The relevant lines in the config file are

```python
c.JupyterHub.ssl_key = '/etc/jupyterhub/srv/server.key'
c.JupyterHub.ssl_cert = '/etc/jupyterhub/srv/server.crt'
c.JupyterHub.port = 8000
```

With a fixed domain name, it should be possible to obtain a certificate from a service such as [Let's Encrypt](https://letsencrypt.org) by installing `certbot` on the system and opening up ports 80 and 443 via the Azure interface. In particular, if JupyterHub is served on 443, then there is no need to include the port (i.e. YOUR_DOMAIN:8000) in the address given to students, or used for authentication below.

## Authentication

Jupyterhub allows multiple methods for user authentication. The default uses the system user controls, so that people log on with the same details that they would to log in to the machine normally. Using an alternative method requires modifying the configuration file that jupyterhub starts from, or adding options on the command line itself. It's possible to access using either GitHub or college credentials using OAuth and the `oauthenticator` package available at https://github.com/jupyterhub/oauthenticator

This package can be pip installed with

    pip3 install oauthenticator

### Microsoft Azure

To use the college log in credentials, it's necessary to create a Microsoft Azure Active Directory App in the Imperial College London directory, via the https://portal.azure.com and the method described at https://www.netiq.com/communities/cool-solutions/creating-application-client-id-client-secret-microsoft-azure-new-portal/.

While creating the app, it's important to note the following information:

 - The ICL Directory ID (from the Azure Active Directory/Properties page).
 - The Azure Application ID (from the Azure Active Directory/App Registrations/App page).

You also need to setup a key from the Azure Active Directory/App Registrations/keys page and to set a reply address of `http://://{your-domain}/hub/oauth_callback` or `https://{your-domain}/hub/oauth_callback`, depending on whether you've got ssl set up.

At this point, you can add the following code block to your `jupyterhub_config.py` file:

```python

import os
from oauthenticator.azuread import LocalAzureAdOAuthenticator
c.JupyterHub.authenticator_class = LocalAzureAdOAuthenticator

c.LocalAzureAdOAuthenticator.tenant_id = os.environ.get('AAD_TENANT_ID')

c.LocalAzureAdOAuthenticator.oauth_callback_url = 'https://YOUR_DOMAIN/hub/oauth_callback'
c.LocalAzureAdOAuthenticator.client_id = APPLICATION_ID
c.LocalAzureAdOAuthenticator.client_secret = APPLICATION_KEY
c.LocalAzureAdOAuthenticator.create_system_users = True
c.LocalAzureAdOAuthenticator.add_user_cmd = ['aaduseradd', '-m', '-o `az ad user show --upn-or-object-id USERNAME --query objectId`']

```

replacing the Ids and domain name as needed. you also need to get the directory ID into the environment, for example by running

```
export AAD_TENANT_ID={the directory string you copied down before}
```

Now you've , you've configured Jupyterhub to use Azure authentication. Unfortunately, the college implementation passes the full name rather than a username, so we need to modify file `oauthenticator/azuread.py`, replacing the line

```python
userdict = {"name": decoded['name']}
```
with

```python
userdict = {"name": decoded['upn']}
```

### Github

Github authentication is very similar to Azure. The URL at which to create an App is https://github.com/settings/applications/new. Here you need to set a name and homepage, as well as the callback URL address of `http://://{your-domain}/hub/oauth_callback` or `https://{your-domain}/hub/oauth_callback`. When the app is created, note the Client ID and Client Secret.

The block in the `jupyterhub_config.py` file just

```python
from oauthenticator.github import LocalGitHubOAuthenticator
c.JupyterHub.authenticator_class = LocalGitHubOAuthenticator
c.LocalGitHubOAuthenticator.create_system_users = True
c.LocalGitHubOAuthenticator.add_user_cmd = ['sudo', 'adduser', '-q', '--gecos', '""', '--disabled-password']

```

however it is necessary to set three environment variables:

```
export GITHUB_CLIENT_ID= { copy from from github app }
export GITHUB_CLIENT_SECRET= { copy from from github app }
export OAUTH_CALLBACK_URL=https://YOUR_DOMAIN/hub/oauth_callback
```
replacing the domain name as appropriate.


### LDAP authentication

## user whitelisting & admin users

The change to the `jupyterhub_config.py` file: to specify a limited set of users who can log in, or to set up admin users with more powers is

```python
# set of users allowed to use the Hub
c.Authenticator.whitelist = {'ggorman', 'mpiggott’}
# set of users who can administer the Hub itself
c.Authenticator.admin_users = {'ggorman'}
```
## Spawning notebook servers

The jupyterhub model has the hub instance create notebook instances for users as and when they are required. Several diffent models are avaiable

 - Local Spawner - the default, uses root privileges to start a new notebook as a local user. The local user may have been created by the authentication process if she did not exist already.
 - Sudo Spawner - For non-root accounts with (passwordless) sudo rights.
 - Docker Spawner - Spins up a docker container for each user. There are options to mount volumes from the host system inside the container if true persistence is required.
 - Kubernetes Spawner - Spin up a docker image in a Kubernetes cluster for each user.

The choice of spawner and the enivronment which it presents is controlled in the config file.

## JupyterHub & Kubernetes

Long term, JupyterHub provides base Docker images of itself, which it should be possible to load onto an Azure AKS Kubernetes cluster. In principle, if this is set up to use the kubernetes spawner for the single user notebooks, then the entire system should scale transparently.

## Initial draft of notes by tmbgreaves for setting up kubernetes+jhub

Initial cluster deployment is documented at https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-deploy-cluster 

Steps followed: 

```
az ad sp create-for-rbac --skip-assignment 
((make a node of the appId and password for later) 
az group create --name=jupyterhub --location=westeurope --output table 
az aks create --resource-group jupyterhub --name jupyterhubaks --node-count 3 --service-principal <hash> --client-secret <hash> --generate-ssh-keys 
```
    
Once the AKS instance completes, set the public DNS within the Azure portal.

Set up a local kube environment in Azure shell (brute force method, overwriting any previous config):

```
rm -rf ~/.kube/ 
az aks get-credentials --resource-group jupyterhub --name jupyterhubaks 
cat > helm-rbac.yaml << EOF 
apiVersion: v1 
kind: ServiceAccount 
metadata: 
  name: tiller 
  namespace: kube-system 
--- 
apiVersion: rbac.authorization.k8s.io/v1beta1 
kind: ClusterRoleBinding 
metadata: 
  name: tiller 
roleRef: 
  apiGroup: rbac.authorization.k8s.io 
  kind: ClusterRole 
  name: cluster-admin 
subjects: 
  - kind: ServiceAccount 
    name: tiller 
    namespace: kube-system 
EOF 
kubectl create -f helm-rbac.yaml 
helm init --service-account tiller 
```

Add the jupyterhub repo:

```
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/ 
helm repo update 
```

Follow instructions from jrper to create a jhub service principal:

* Login to the portal and go to Azure Active Directory > App registrations > New application registrations
* give a name and (any) url 
* Create and note the Application ID number (which is the AAD_CLIENT_ID above) 
* Then on the app settings>Keys and in the passwords box add a description and duration for a client secret. Click save and note the value carefully (this is AAD_CLIENT_SECRET) 
* Next on the app setttings>Reply urls add the jupyterhub callback url, which is of the form https://<your host name> :<your port>/hub/oauth_callback (this is the OAUTH_CALLBACK_URL) 
* Finally on Azure Active Directory > Properties, note the Directory ID for imperial college london (this is the AAD_TENANT_ID above)

Now create the configuration for jhub:

```
cat >> config.yaml << EOF 
proxy: 
  secretToken: "<newly-generated-64char-hash>" 
  https: 
    hosts: 
      - "ese-jhub.westeurope.cloudapp.azure.com" 
    letsencrypt: 
      contactEmail: xxx.xxx@imperial.ac.uk 
    
hub: 
  extraEnv: 
    AAD_TENANT_ID: "<hash>" 
  image: 
    name: tmbgreaves/jupyterhub-k8s 
    tag: '0.7.0-usernamehack' 

auth: 
  type: custom 
  custom: 
    className: 'oauthenticator.azuread.AzureAdOAuthenticator' 
    config: 
      client_id: "<hash>" 
      client_secret: "<hash>" 
      oauth_callback_url: "https://ese-jhub.westeurope.cloudapp.azure.com/hub/oauth_callback" 
      tenant_id: "<hash>" 
  admin:
    users:
      - "tmb1"
      - "jrper"
      - "skramer"
      - "nbarral"
      - "ggorman"

singleuser:
  image:
    name: stephankramer/jhub-notebook-firefox
    tag: "20180927"

EOF
```

And deploy:

```
helm upgrade --install jupyterhub jupyterhub/jupyterhub --namespace jupyterhub --version 0.7.0   --values config.yaml 
```
 


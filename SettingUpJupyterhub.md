# Setting up JupyterHub on a local or cloud machine

See also  https://github.com/jupyterhub/jupyterhub-tutorial/blob/master/JupyterHub.pdf for a more general discussion.

## Installing Jupyterhub

For a generic Ubuntu installation you can use:

    sudo python3 -m pip install jupyterhub
    npm install -g configurable-http-proxy

Or for one with conda installed:

    conda install -c conda-forge jupyterhub
    conda install notebook

## SSL certificates

## Authentication

Jupyterhub allows multiple methods for user authentication. The default uses the system user controls, so that people log on with the same details that they would to log in to the machine normally. Using an alternative method requires modifying the configuration file that jupyterhub starts from, or adding options on the command line itself. It's possible to access using either GitHub or college credentials using OAuth and the `oauthenticator` package available at https://github.com/jupyterhub/oauthenticator

This package can be pip installed with

    pip3 install oauthenticator

### Microsoft Azure

To use the college log in credentials, it's necessary to create a Microsoft Azure Active Directory App in the Imperial College London directory, via the https://portal.azure.com and the method described at https://www.netiq.com/communities/cool-solutions/creating-application-client-id-client-secret-microsoft-azure-new-portal/.

While creating the app, it's important to note the following information:

 - The ICL Directory ID (from the Azure Active Directory/Properties page).
 - The Azure Application ID (from the Azure Active Directory/App Registrations/App page).

You also need to setup a key from the Azure Active Directory/App Registrations/keys page and to set a reply address of `http://://{your-domain}/hub/oauth_callback` or `https://{your-domain}/hub/oauth_callback`

At this point, you can add the following code block to your `jupyterhub_config.py` file:

```python

import os
from oauthenticator.azuread import LocalAzureAdOAuthenticator
c.JupyterHub.authenticator_class = LocalAzureAdOAuthenticator

c.LocalAzureAdOAuthenticator.tenant_id = os.environ.get('AAD_DIRECTORY_ID')

c.LocalAzureAdOAuthenticator.oauth_callback_url = 'https://YOUR_DOMAIN/hub/oauth_callback'
c.LocalAzureAdOAuthenticator.client_id = APPLICATION_IP
c.LocalAzureAdOAuthenticator.client_secret = APPLICATION_KY
c.LocalAzureAdOAuthenticator.create_system_users = True
c.LocalAzureAdOAuthenticator.add_user_cmd = ['adduser', '-q', '--gecos', '""', '--disabled-password']

```

replacing the Ids and domain name as needed. you also need to get the directory ID into the environment, for example by running

```
export AAD_DIRECTORY_ID={the string you copied down before}
```

At this point, you've configured Jupyterhub to use Azure authentication. Unfortunately, the college implementation passes the full name rather than a username, so we need to modify file `oauthenticator/azuread.py`, replacing the line

```python
userdict = {"name": decoded['name']}
```
with

```python
userdict = {"name": decoded['upn'].split('@')[0]}
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
c.Authenticator.whitelist = {'minrk', 'takluyver’}
# set of users who can administer the Hub itself
c.Authenticator.admin_users = {'mink'}
```

## Spawning notebook servers
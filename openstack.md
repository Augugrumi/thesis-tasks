---
description: Aka the gate to hell
---

# Openstack

## Tacker

### Set up default vim

In order to add new NFV, you need to have a VIM already set up. To set it up, you first need to login as `nfv_user` \(the default password is `devstack`\), then download the v3 openrc script, and then from the console include it \(with `source <namefile>`\). 

{% hint style="info" %}
The openrc file should look like this:

{% code-tabs %}
{% code-tabs-item title="openrc.sh" %}
```bash
#!/usr/bin/env bash
# To use an OpenStack cloud you need to authenticate against the Identity
# service named keystone, which returns a **Token** and **Service Catalog**.
# The catalog contains the endpoints for all services the user/tenant has
# access to - such as Compute, Image Service, Identity, Object Storage, Block
# Storage, and Networking (code-named nova, glance, keystone, swift,
# cinder, and neutron).
#
# *NOTE*: Using the 3 *Identity API* does not necessarily mean any other
# OpenStack API is version 3. For example, your cloud provider may implement
# Image API v1.1, Block Storage API v2, and Compute API v2.0. OS_AUTH_URL is
# only for the Identity API served through keystone.
export OS_AUTH_URL=http://192.168.29.22/identity/v3
# With the addition of Keystone we have standardized on the term **project**
# as the entity that owns the resources.
export OS_PROJECT_ID=ea4b42df595f47a58731dc8672e2c19b
export OS_PROJECT_NAME="nfv"
export OS_USER_DOMAIN_NAME="Default"
if [ -z "$OS_USER_DOMAIN_NAME" ]; then unset OS_USER_DOMAIN_NAME; fi
export OS_PROJECT_DOMAIN_ID="default"
if [ -z "$OS_PROJECT_DOMAIN_ID" ]; then unset OS_PROJECT_DOMAIN_ID; fi
# unset v2.0 items in case set
unset OS_TENANT_ID
unset OS_TENANT_NAME
# In addition to the owning entity (tenant), OpenStack stores the entity
# performing the action as the **user**.
export OS_USERNAME="nfv_user"
# With Keystone you pass the keystone password.
echo "Please enter your OpenStack Password for project $OS_PROJECT_NAME as user $OS_USERNAME: "
read -sr OS_PASSWORD_INPUT
export OS_PASSWORD=$OS_PASSWORD_INPUT
# If your configuration has multiple regions, we set that information here.
# OS_REGION_NAME is optional and only valid in certain environments.
export OS_REGION_NAME="RegionOne"
# Don't leave a blank variable, unset it if it was empty
if [ -z "$OS_REGION_NAME" ]; then unset OS_REGION_NAME; fi
export OS_INTERFACE=public
export OS_IDENTITY_API_VERSION=3
```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endhint %}

The script will ask for the NFV user password, type it again.  
After this, you need to add a VIM site from a YAML file. A tipical YAML configuration is:

{% code-tabs %}
{% code-tabs-item title="vim\_config.yaml" %}
```yaml
auth_url: 'http://192.168.29.22/identity'
username: 'nfv_user'
password: 'devstack'
project_name: 'nfv'
project_domain_name: 'Default'
user_domain_name: 'Default'
cert_verify: 'True'
```
{% endcode-tabs-item %}
{% endcode-tabs %}

To add the VIM site to you Openstack installation, type the following command in your console:

```bash
tacker vim-register --description 'test' --config-file vim_config.yaml Site1
```

After that, you should have the VIM in your list, like in the picture below.

![](.gitbook/assets/image%20%281%29.png)




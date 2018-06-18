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
*todo* to insert here
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




---
description: Aka the gate to hell
---

# Openstack

## Preamble

[Openstack](https://www.openstack.org/) is a useful tool that allows the creation of in-house cloud solutions. In our case, Openstack is fundamental because it's used by the VIM \(see [Openbaton installation instructions](https://augugrumi.gitbook.io/thesisnotes/~/edit/drafts/-LFr0kzwrv6rjp5HpaWM/openstack)\) to launch instances and deploy services, other than create internal networking and routing.

## Installation

The Openstack installation is not an easy one, particularly for newcomers, that have to deal with a great set of tools and not-always-clear instruction. Here we won't try to explain you how to perfectly install an Openstack instance, but we'll describe how we managed to get a single-node deployment up and running.

First of all, you need a machine with _at least_ 16GB of ram and 50GB of HD space.

### Devstack

[Devstack](https://docs.openstack.org/devstack/latest/) \(god bless\) allows to deploy a developer version of Openstack in a single node, and it makes modules activation and installation a piece of cake. In our case, we need to have an active version of:

* Keystone \(for identity management\)
* Object storage
* Compute
* Tacker \(not mandatory if you're gonna use Openbaton as VIM\)

Using Devstack is a brain-dead operation, but configuring it, oh boy, is not an easy one. In particular, the time required to install Openstack ranges from 20 to 1 hour, and every time your configuration is wrong you have to start all over again from scratch. For reference, after many tries, our `local.conf` file was:

{% code-tabs %}
{% code-tabs-item title="local.conf" %}
```bash
[[local|localrc]]
############################################################
# Customize the following HOST_IP based on your installation
############################################################
HOST_IP=<your ip address>

ADMIN_PASSWORD=Password1
MYSQL_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
SERVICE_TOKEN=$ADMIN_PASSWORD

############################################################
# Customize the following section based on your installation
############################################################

# Pip
PIP_USE_MIRRORS=False
USE_GET_PIP=1

#OFFLINE=False
#RECLONE=True

# Logging
LOGFILE=$DEST/logs/stack.sh.log
VERBOSE=True
ENABLE_DEBUG_LOG_LEVEL=True
ENABLE_VERBOSE_LOG_LEVEL=True

# Neutron ML2 with OpenVSwitch
Q_PLUGIN=ml2
Q_AGENT=openvswitch

SWIFT_REPLICAS=1
SWIFT_HASH=66a3d6b56c1f479c8b4e70ab5c2000f5
FLOATING_RANGE=<your ip address.224/27>
FLOAT_INTERFACE=<select your interface where expose the service>

enable_service s-proxy s-object s-container s-account
enable_service h-eng h-api h-api-cfn h-api-cw

# Disable security groups
Q_USE_SECGROUP=False
LIBVIRT_FIREWALL_DRIVER=nova.virt.firewall.NoopFirewallDriver

# Enable heat, networking-sfc, barbican and mistral
enable_plugin heat https://git.openstack.org/openstack/heat stable/queens
enable_plugin networking-sfc git://git.openstack.org/openstack/networking-sfc stable/queens
enable_plugin barbican https://git.openstack.org/openstack/barbican stable/queens
enable_plugin mistral https://git.openstack.org/openstack/mistral stable/queens

# Ceilometer
#CEILOMETER_PIPELINE_INTERVAL=300
enable_plugin ceilometer https://git.openstack.org/openstack/ceilometer stable/queens
enable_plugin aodh https://git.openstack.org/openstack/aodh stable/queens

# Tacker
enable_plugin tacker https://git.openstack.org/openstack/tacker stable/queens

enable_service n-novnc
enable_service n-cauth

disable_service tempest

# Enable Kubernetes and kuryr-kubernetes
#KUBERNETES_VIM=True
#NEUTRON_CREATE_INITIAL_NETWORKS=False
#enable_plugin kuryr-kubernetes https://git.openstack.org/openstack/kuryr-kubernetes stable/queens
#enable_plugin neutron-lbaas git://git.openstack.org/openstack/neutron-lbaas stable/queens
#enable_plugin devstack-plugin-container https://git.openstack.org/openstack/devstack-plugin-container stable/queens

[[post-config|/etc/neutron/dhcp_agent.ini]]
[DEFAULT]
enable_isolated_metadata = True
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="warning" %}
Be aware to change the values inside the `<` and `>` brackets.
{% endhint %}

{% hint style="info" %}
In our case, we were able to develop a simple wrapper around the `stack.sh`command \(necessary to launch Openstack installation\), that you can find in [our repository](https://github.com/Augugrumi/vagrantfiles/blob/master/openstack/bootstrap.sh), but, for reference, we copy it here:

{% code-tabs %}
{% code-tabs-item title="bootstrap.sh" %}
```bash
#!/bin/bash

# Author: Davide Polonio <poloniodavide@gmail.com>
# License: GPLv3+

function check () {
    if [ "$1" -ne 0 ]
    then
        msg err "$2"
        exit 1
    fi
}

function msg () {
    # 3 type of messages:
    # - info
    # - warn
    # - err
    local color=""
    local readonly default="\033[m" #reset
    if [ "$1" = "info" ]
    then
        color="\033[0;32m" #green
    elif [ "$1" = "warn" ]
    then
        color="\033[1;33m" #yellow
    elif [ "$1" = "err" ]
    then
        color="\033[0;31m" #red
    fi

    echo -e "$color==> $2$default"
}

function getUbuntuVersion () {
    echo $(lsb_release -r | cut -f2)
}

function openstackWizard () {
    sudo apt-get update
    sudo apt-get upgrade -y
    sudo apt-get install -y python-systemd
    sudo apt-get autoclean
    sudo apt-get autoremove -y

    cd $HOME
    rm -rf devstack/
    git clone https://git.openstack.org/openstack-dev/devstack -b "$1" --depth=1
    check $? "Failed to clone openstack repo"
    cd devstack

    sudo mkdir /logs
    sudo chown -R "$(whoami)":"$(whoami)" /logs
    mkdir logs

    local readonly ADMIN_PASSWORD="Password1"
    local readonly myIpAddress="$(ip a show $2 | grep inet | head -n1 | cut -d" " -f6 | cut -d"/" -f1)"
    cat <<EOF > local.conf
[[local|localrc]]
############################################################
# Customize the following HOST_IP based on your installation
############################################################
HOST_IP=$myIpAddress

ADMIN_PASSWORD=$ADMIN_PASSWORD
MYSQL_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
SERVICE_TOKEN=$ADMIN_PASSWORD

############################################################
# Customize the following section based on your installation
############################################################

# Pip
PIP_USE_MIRRORS=False
USE_GET_PIP=1

#OFFLINE=False
#RECLONE=True

# Logging
LOGFILE=$DEST/logs/stack.sh.log
VERBOSE=True
ENABLE_DEBUG_LOG_LEVEL=True
ENABLE_VERBOSE_LOG_LEVEL=True

# Neutron ML2 with OpenVSwitch
Q_PLUGIN=ml2
Q_AGENT=openvswitch

SWIFT_REPLICAS=1
SWIFT_HASH=66a3d6b56c1f479c8b4e70ab5c2000f5
FLOATING_RANGE=$(echo "$myIpAddress" | cut -d"." -f4 --complement).224/27
FLOAT_INTERFACE=$2

enable_service s-proxy s-object s-container s-account
enable_service h-eng h-api h-api-cfn h-api-cw

# Disable security groups
Q_USE_SECGROUP=False
LIBVIRT_FIREWALL_DRIVER=nova.virt.firewall.NoopFirewallDriver

# Enable heat, networking-sfc, barbican and mistral
enable_plugin heat https://git.openstack.org/openstack/heat stable/queens
enable_plugin networking-sfc git://git.openstack.org/openstack/networking-sfc stable/queens
enable_plugin barbican https://git.openstack.org/openstack/barbican stable/queens
enable_plugin mistral https://git.openstack.org/openstack/mistral stable/queens

# Ceilometer
#CEILOMETER_PIPELINE_INTERVAL=300
enable_plugin ceilometer https://git.openstack.org/openstack/ceilometer stable/queens
enable_plugin aodh https://git.openstack.org/openstack/aodh stable/queens

# Tacker
enable_plugin tacker https://git.openstack.org/openstack/tacker stable/queens

enable_service n-novnc
enable_service n-cauth

disable_service tempest

# Enable Kubernetes and kuryr-kubernetes
#KUBERNETES_VIM=True
#NEUTRON_CREATE_INITIAL_NETWORKS=False
#enable_plugin kuryr-kubernetes https://git.openstack.org/openstack/kuryr-kubernetes stable/queens
#enable_plugin neutron-lbaas git://git.openstack.org/openstack/neutron-lbaas stable/queens
#enable_plugin devstack-plugin-container https://git.openstack.org/openstack/devstack-plugin-container stable/queens

[[post-config|/etc/neutron/dhcp_agent.ini]]
[DEFAULT]
enable_isolated_metadata = True
EOF

    msg info "Launching devstack installation..."
    sleep 3 # Calm before the storm...
    ./stack.sh
    check $? "Something bad happened during devstack installation! I'm sorry :("
    cd -
    cd horizon/ && python manage.py compress && cd -
    check $? "Failed to recreate horizon cache!"
    sudo service apache2 restart
    msg info "Devstack installation complete! Enjoy!"
}

function main () {

    local readonly LAUNCH_USERNAME="$(whoami)"
    local readonly LAUNCH_USERNAME_HOME=$HOME
    local readonly NEW_USERNAME="stack"
    local readonly NEW_USERNAME_HOME="/opt/stack"

    local branch="master"
    local linkinterface=""
    local install=1
    while getopts ":i :b: :l:" opt; do
        case $opt in
            i)
                msg info "Installing Openstack with user $(whoami)..."
                install=0
                ;;
            b)
                msg info "Setting branch to $OPTARG"
                branch="$OPTARG"
                ;;
            l)
                msg info "Interface link set to $OPTARG"
                linkinterface="$OPTARG"
                ;;
            \?)
                msg err "Invalid option: -$OPTARG" >&2
                exit 1
                ;;
            :)
                msg err "Option -$OPTARG requires an argument." >&2
                exit 1
                ;;
        esac
    done

    if [ "$install" -eq 0 ]
    then
	if [ "$linkinterface" != "" ]
	then
	    openstackWizard "$branch" "$linkinterface"
	else
	    msg err "You MUST set a pastebin url for your local config"
	fi
    else
	if [ "$(getUbuntuVersion)" != "16.04" ]
	then
            msg err "This script supports only Ubuntu 16.04"
	fi
	
	if [ "$(pwd)" != "$HOME" ]
	then
	    msg info "Copying the script in the right $HOME"
	    cp "$0" "$HOME/$(basename $0)"
	    check $? "Failed to copy the script in the right location"
	    rm $0
	    cd $HOME
	    chmod +x "$(basename $0)"
	    ./$(basename $0) $@
	fi
	msg info "Launching installation as $LAUNCH_USERNAME"
        echo "$LAUNCH_USERNAME ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/$LAUNCH_USERNAME
        sudo useradd -s /bin/bash -d $NEW_USERNAME_HOME -m $NEW_USERNAME
        echo "$NEW_USERNAME ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/$NEW_USERNAME
	sudo -p "Restarting as stack, please type your credentials\n" su -c "$LAUNCH_USERNAME_HOME/bootstrap.sh -b $branch -l $linkinterface -i" $NEW_USERNAME
    fi
}

main $@

```
{% endcode-tabs-item %}
{% endcode-tabs %}

You need to pass two arguments to this script, that are:

* b: the branch of the openstack version you want to use
* l: the network link where openstack will listen for incoming requests

To sum up, here there is a valid example:  
`./bootstrap.sh -b stable/queens -l ens3`

After that, you should able to login using `admin` and `Password1` as credentials to your new, shining, Openstack installation. The script automatically adds a `stack` user into the system, where it performs the devstack installation. You should be able to log in this user simply typing:   
`sudo su -l stack`
{% endhint %}

## Configurations

### Tacker

If you used our script or our configuration, you should have Tacker installed and working. Here you can find a guide to how setup a default VIM and how to launch VNF instances.

### Set up default VIM

In order to add new VNF, you need to have a VIM already set up. To set it up, you first need to login as `nfv_user` \(the default password is `devstack`\), then download the v3 `openrc` script, and then from the console include it \(with `source <namefile>`\).

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


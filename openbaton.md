---
description: '''Cuz we don''t like Tacker'
---

# Openbaton

## Preamble

Openbaton is an alternative VIM that can be linked with Openstack. After some developement, we had to change from Tacker to this, so here we are.

## Installations

### On "bare" OS

There are two ways to install Openbaton from a bare OS: you can install it interactively or settings all the options in a config file, and then let the installation procedure run alone in the darkness of your shell.

#### Interactive installation

Openbaton installation is pretty straight-forward. Open a terminal and execute the wizard made by the Openbaton team:

```bash
sh <(curl -s http://get.openbaton.org/bootstrap) release
```

Then follow the instruction in the screen and enjoy your Openstack installation!

#### Installation with config. file

The configuration file is simply a place where you set all the variables in one place, so that the installer won't ask for additional information during the installation. This, combined with screen, allows to run faster installations.

{% code-tabs %}
{% code-tabs-item title="conf" %}
```bash
# This is a configuration file for the Open Baton bootstrap. 
# It allows you to specify the parameters needed during the bootstrap for the configuration of the Open Baton projects.
#
# Usage: sh <(curl -s http://get.openbaton.org/bootstrap) [help | clean | enable-persistence | configure-remote-rabbitmq] | [[upgrade] [--openbaton-components=<all | openbaton-xxx,openbaton-yyy,...>]] | [[release | develop] [--openbaton-bootstrap-version=X.Y.Z (with X.Y.Z >= 3.2.0)] [--config-file=<absolute path to configuration file>]]
#
# IMPORTANT NOTE: Avoid spaces before and after the '=': i.e. a parameter needs to be specified as 'parameter=value'


#################
#### General ####
#################

openbaton_bootstrap_version=latest                          # Deafult is 'latest' therefore if left empty ('openbaton_bootstrap_version=') or commented the bootstrap used will be the latest. Use format X.Y.Z for a specific version: the oldest VERSION installable is 3.2.0
openbaton_installation_mode=noninteractive                  # Deafult is 'interactive' therefore if left empty ('openbaton_installation_mode=') or commented the installation will be interactive. Use 'noninteractive' or 'Noninteractive' for a not interactive installation
openbaton_component_autostart=true                          # Deafult is 'true' therefore if left empty ('openbaton_component_autostart=') or commented the debian component will start automatically at the end of the installation


##############
#### NFVO ####
##############

rabbitmq_broker_ip=<set yout ip>                                # Default is 'localhost' therefore if left empty ('rabbitmq_broker_ip=') or commented the 'localhost' value will be used
rabbitmq_management_port=15672                              # Default is '15672' therefore if left empty ('rabbitmq_management_port=') or commented the '15672' value will be used
openbaton_nfvo_ip=localhost                                 # Default is 'localhost' therefore if left empty ('openbaton_nfvo_ip=') or commented the 'localhost' value will be used
openbaton_admin_password=Password1                          # Default is 'openbaton' therefore if left empty ('openbaton_admin_password=') or commented the 'openbaton' value will be used

https=yes                                                    # Default is 'NO' therefore if left empty ('https=') or commented the HTTPS will NOT be enabled
mysql=yes                                                   # Default is 'YES' therefore if left empty ('mysql=') or commented the MYSQL DB will be installed and the Open Baton persistence will be enabled
mysql_root_password=Password1                                    # Default is 'root' therefore if left empty ('mysql_root_password=') or commented the 'root' value will be used (NOTE: you should insert here the actual mysql root password if mysql is already installed in the system)
openbaton_nfvo_mysql_user=admin                             # Default is 'admin' therefore if left empty ('openbaton_nfvo_mysql_user=') or commented the 'admin' value will be used
openbaton_nfvo_mysql_user_password=Password1                 # Default is 'changeme' therefore if left empty ('openbaton_nfvo_mysql_user_password=') or commented the 'changeme' value will be used


##########################################
#### Open Baton additional components ####
##########################################

openbaton_plugin_vimdriver_test=yes                         # Default is 'YES' therefore if left empty ('openbaton_plugin_vimdriver_test=') or commented the 'openbaton_plugin_vimdriver_test' driver will be installed (this option is valid only for develop installation)
openbaton_plugin_vimdriver_openstack=yes                    # Default is 'YES' therefore if left empty ('openbaton_plugin_vimdriver_openstack=') or commented the 'openbaton_plugin_vimdriver_openstack' debian package will be installed
openbaton_plugin_monitoring_zabbix=yes                       # Default is 'NO' therefore if left empty ('openbaton_plugin_monitoring_zabbix=') or commented the 'openbaton_plugin_monitoring_zabbix' debian package will be installed
openbaton_vnfm_generic=yes                                  # Default is 'YES' therefore if left empty ('openbaton_vnfm_generic=') or commented the 'openbaton_vnfm_generic' debian package will be installed
openbaton_fms=yes                                           # Default is 'NO' therefore if left empty ('openbaton_fms=') or commented the 'openbaton_fms' debian package will NOT be installed
openbaton_ase=yes                                           # Default is 'NO' therefore if left empty ('openbaton_ase=') or commented the 'openbaton_ase' debian package will NOT be installed
openbaton_nse=yes                                           # Default is 'NO' therefore if left empty ('openbaton_nse=') or commented the 'openbaton_nse' debian package will NOT be installed

# NOTE: The VERSION is to be interptreted as "debian package version" or "source TAG" respectively for RELEASE / DEVELOP installation
# Possible values are: 'latest' (only for RELEASE), 'develop' (only for DEVELOP), 'X.Y.Z' (for both types of installation)
openbaton_nfvo_version=5.2.1                                # Default is 'latest' / 'develop' therefore if left empty ('openbaton_nfvo_version=') or commented the 'latest' debian version / the 'develop' TAG will be installed. NOTE: Check the list of available tags at: https://github.com/openbaton/generic-vnfm/tags - The oldest VERSION installable is 3.2.0
openbaton_plugin_vimdriver_test_version=5.2.1               # Default is 'latest' / 'develop' therefore if left empty ('openbaton_plugin_vimdriver_test_version=') or commented the 'latest' debian version / the 'develop' TAG will be installed. NOTE: Check the list of available tags at: https://github.com/openbaton/openstack4j-plugin/tags - The oldest VERSION installable is 3.2.0
openbaton_plugin_vimdriver_openstack_version=5.2.1          # Default is 'latest' / 'develop' therefore if left empty ('openbaton_plugin_vimdriver_openstack_version=') or commented the 'latest' debian version / the 'develop' TAG will be installed. NOTE: Check the list of available tags at: https://github.com/openbaton/openstack4j-plugin/tags - The oldest VERSION installable is 3.2.0
openbaton_plugin_monitoring_zabbix_version=5.2.0            # Default is 'latest' / 'develop' therefore if left empty ('openbaton_plugin_monitoring_zabbix_version=') or commented the 'latest' debian version / the 'develop' TAG will be installed. NOTE: Check the list of available tags at: https://github.com/openbaton/zabbix-plugin/tags - The oldest VERSION installable is 3.2.0
openbaton_vnfm_generic_version=5.2.1                        # Default is 'latest' / 'develop' therefore if left empty ('openbaton_vnfm_generic_version=') or commented the 'latest' debian version / the 'develop' TAG will be deployed. NOTE: Check the list of available tags at: https://github.com/openbaton/generic-vnfm/tags - The oldest VERSION installable is 3.2.0
openbaton_fms_version=1.4.2                                 # Default is 'latest' / 'develop' therefore if left empty ('openbaton_fms_version=') or commented the 'latest' debian version / the 'develop' TAG will be deployed. NOTE: Check the list of available tags at: https://github.com/openbaton/fm-system/tags - The oldest VERSION installable is 1.2.1
openbaton_ase_version=latest                                 # Default is 'latest' / 'develop' therefore if left empty ('openbaton_ase_version=') or commented the 'latest' debian version / the 'develop' TAG will be deployed. NOTE: Check the list of available tags at: https://github.com/openbaton/autoscaling-engine/tags - The oldest VERSION installable is 1.2.2
openbaton_nse_version=1.1.2                                 # Default is 'latest' / 'develop' therefore if left empty ('openbaton_nse_version=') or commented the 'latest' debian version / the 'develop' TAG will be deployed. NOTE: Check the list of available tags at: https://github.com/openbaton/network-slicing-engine/tags - The oldest VERSION installable is 1.1.2


#############
#### FMS ####
#############

openbaton_fms_mysql_user=fmsuser                            # Default is 'fmsuser' therefore if left empty ('mysql_user=') or commented the 'admin' value will be used
openbaton_fms_mysql_user_password=Password1                  # Default is 'changeme' therefore if left empty ('mysql_user_password=') or commented the 'changeme' value will be used


######################################################
#### Zabbix Plugin (required by FMS, ASE and NSE) ####
######################################################

# NOTE: Currently the ZABBIX configuration parameters are supported only for the RELEASE installation 
zabbix_plugin_ip=                                           # Default is 'localhost' therefore if left empty ('zabbix_plugin_ip=') or commented the 'admin' value will be used
zabbix_server_ip=                                           # Default is 'localhost' therefore if left empty ('zabbix_server_ip=') or commented the 'admin' value will be used
zabbix_user=                                                # Default is 'Admin' therefore if left empty ('zabbix_user=') or commented the 'Admin' value will be used
zabbix_user_password=Password1                                       # Default is 'zabbix' therefore if left empty ('zabbix_user_password=') or commented the 'zabbix' value will be used

```
{% endcode-tabs-item %}
{% endcode-tabs %}

At this point, to run the installation procedure type in a terminal:

```bash
sh <(curl -s http://get.openbaton.org/bootstrap) release --config-file=`pwd`/conf
```

Then enjoy the shell doing fancy stuff! 😎

### Using docker-compose

Sometimes installing Openbaton directly on the OS is not viable, specially if your messing with the it. Every time you break it you have to install everything from scratch, and this is a waste of time. Fortunately, docker come to rescue! From the Openbaton team [we've stolen a \(for us\) non-working docker-compose,](https://github.com/openbaton/bootstrap/blob/1b20e2b4ce1ab6e64b110816a7e54f5c0bca48be/distributions/docker/compose/full-compose-with-zabbix.yml) and we've fixed it making it a great tool to deploy Openbaton in seconds. From here you can stole our version:

{% code-tabs %}
{% code-tabs-item title="docker-compose.yaml" %}
```yaml
version: '3'
services:
  nfvo:
    image: openbaton/nfvo:latest
    depends_on:
      - rabbitmqbroker
      - nfvodatabase
      - zabbixserver
    restart: always
    environment: 
      - NFVO_RABBIT_BROKERIP=${HOST_IP} # for use in userdata.sh in vnfm-generic
      - NFVO_MONITORING_IP=${HOST_IP}
      - NFVO_QUOTA_CHECK=false
      - NFVO_PLUGIN_INSTALLATION-DIR=/dev/null
      - SPRING_RABBITMQ_HOST=rabbitmqbroker
      - SPRING_DATASOURCE_URL=jdbc:mysql://nfvodatabase:3306/openbaton
      - SPRING_DATASOURCE_DRIVER-CLASS-NAME=org.mariadb.jdbc.Driver
      - SPRING_JPA_DATABASE-PLATFORM=org.hibernate.dialect.MySQLDialect
      - SPRING_JPA_HIBERNATE_DDL-AUTO=update
    ports:
      - "8080:8080"
  vnfmgeneric:
    image: openbaton/vnfm-generic:latest
    depends_on:
      - nfvo
    restart: always
    environment: 
      - VNFM_RABBITMQ_BROKERIP=rabbitmqbroker
  pluginvimdrivertest:
    image: openbaton/plugin-vimdriver-test:5.1.0
    depends_on:
      - nfvo
    restart: always
    environment:
      - RABBITMQ=rabbitmqbroker
  pluginvimdriveropenstack4j:
    image: openbaton/plugin-vimdriver-openstack-4j:latest
    depends_on:
      - nfvo
    restart: always
    environment:
      - RABBITMQ=rabbitmqbroker
  pluginmonitoringzabbix:
    image: openbaton/plugin-monitoring-zabbix:5.1.1
    depends_on:
      - nfvo
    restart: always
    environment:
      - ZABBIX_PLUGIN_IP=${HOST_IP}
      - FAULTS_CONSUMER_ENDPOINT=http://faultmanagementsystem:9000/alarm/vr
      - ZABBIX_HOST=zabbixwebapachemysql
      - RABBITMQ_BROKERIP=rabbitmqbroker
      - ZABBIX_ENDPOINT=/api_jsonrpc.php
      - ZABBIX_PORT=80
    ports:
      - "8010:8010"
  faultmanagementsystem:
    image: openbaton/fms:latest
    depends_on:
      - nfvo
      - pluginmonitoringzabbix
      - fmsdatabase
    restart: always
    environment:
      - SPRING_RABBITMQ_HOST=rabbitmqbroker
      - NFVO_IP=nfvo
      - PLUGIN_IP=zabbix-plugin
      - SPRING_DATASOURCE_URL=jdbc:mysql://fmsdatabase:3306/faultmanagement
      - MYSQL_DATABASE=faultmanagement
      - MYSQL_USER=fmsuser
      - MYSQL_PASSWORD=Password1
  autoscalingengine:
    image: openbaton/ase:latest
    depends_on:
      - nfvo
      - pluginmonitoringzabbix
    restart: always
    environment:
      - ASE_RABBITMQ_BROKERIP=rabbitmqbroker
      - ASE_SERVER_IP=autoscalingengine
      - NFVO_IP=nfvo
      - PLUGIN_IP=zabbix-plugin
  networkslicingengine:
    image: openbaton/nse:latest
    depends_on:
      - nfvo
    restart: always
    environment:
      - RABBITMQ_HOST=rabbitmqbroker
      - NFVO_IP=nfvo
  rabbitmqbroker:
    image: rabbitmq:3-management-alpine
    restart: always
    hostname: openbaton-rabbitmq
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=openbaton
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitdata:/var/lib/rabbitmq

  zabbixserver:
    image: zabbix/zabbix-server-mysql:latest
    ports:
      - "10051:10051"
    volumes:
      - zbxalert:/usr/lib/zabbix/alertscripts
    environment:
      - DB_SERVER_HOST=zabbixdatabase
      - MYSQL_DATABASE=zabbix
      - MYSQL_USER=zabbix
      - MYSQL_PASSWORD=Password1
      - MYSQL_ROOT_PASSWORD=Password1

  zabbixwebapachemysql:
    image: zabbix/zabbix-web-apache-mysql
    depends_on:
      - zabbixdatabase
      - zabbixserver
    ports:
      - "80:80"
    environment:
      - DB_SERVER_HOST=zabbixdatabase
      - ZBX_SERVER_HOST=zabbixserver
      - MYSQL_DATABASE=zabbix
      - PHP_TZ=Europe/Rome
      - MYSQL_USER=zabbix
      - MYSQL_PASSWORD=Password1
      - MYSQL_ROOT_PASSWORD=Password1
        
  nfvodatabase:
    image: mysql:5.7
    restart: always

    environment:
      - MYSQL_ALLOW_EMPTY_PASSWORD=true
      - MYSQL_DATABASE=openbaton
      - MYSQL_USER=admin
      - MYSQL_PASSWORD=changeme
    volumes:
      - dbdata_nfvo:/var/lib/mysql
  fmsdatabase:
    image: mysql:5.7
    restart: always
    environment:
      - MYSQL_ALLOW_EMPTY_PASSWORD=true
      - MYSQL_DATABASE=faultmanagement
      - MYSQL_USER=fmsuser
      - MYSQL_PASSWORD=changeme
    volumes:
      - dbdata_fms:/var/lib/mysql

  zabbixdatabase:
    image: mysql:5.7
    command: [mysqld, --character-set-server=utf8, --collation-server=utf8_bin]
    restart: always
    environment:
      - MYSQL_DATABASE=zabbix
      - MYSQL_USER=zabbix
      - MYSQL_PASSWORD=Password1
      - MYSQL_ROOT_PASSWORD=Password1 
    volumes:
      - dbdata_zabbix:/var/lib/mysql 

volumes:
  rabbitdata:
  zbxalert:
  dbdata_fms:
  dbdata_nfvo:
  dbdata_zabbix:

```
{% endcode-tabs-item %}
{% endcode-tabs %}

Launching this configuration is very simple. Put it in a folder, cd in it and type in a terminal:

```bash
env HOST_IP=<you machine ip> docker-compose -f full-compose.yaml up -d
```

You can watch the Openbaton log typing:

```bash
docker-compose -f full-compose.yaml logs -f
```

Please note that the startup phase could require some time. On top of that, to have decent performance we recommend you to use a machine with at least 16GB of RAM.

### Link Openbaton to Openstack

In Openbaton the space where VNFs are deployed is called POP \(point of presence\). There can be multiple POPs, and you have to link them. Through the user interface, go to Manage PoPs -&gt; PoPs instances add one. You can add a PoP filling the forms or uploading a JSON. If you choose the later one, you can copy-paste the following:

```javascript
{  
   "name":"openstackvnf",
   "authUrl":"http://<openstack ip>/identity/v3/",
   "tenant":"<project id>",
   "username":"admin",
   "password":"Password1",
   "keyPair":"openbaton",
   "securityGroups":[  
      "default"
   ],
   "type":"openstack",
   "location":{  
      "name":"Heart of Gold",
      "latitude":"XX.XXXXX",
      "longitude":"XX.XXXXX"
   }
}

```

Where in `authUrl` you have to insert your openstack keystone API IP \(if you followed our openstack installation, you only need to change your IP, otherwise you have to set up a proper path to your API\), and in the `tenant` field you have to provide a valid project id \(if you're using API version 3\) where the VNFs will be deployed. The `location` and the `securityGroups` fields are not mandatory.

Additional documentation \(since we haven't invented the wheel\) can be found at the following links:

{% embed data="{\"url\":\"https://openbaton.github.io/documentation/pop-registration/\",\"type\":\"link\",\"title\":\"OpenBaton Documentation\",\"icon\":{\"type\":\"icon\",\"url\":\"https://openbaton.github.io/documentation/pop-registration/favicon.ico\",\"aspectRatio\":0}}" %}

{% embed data="{\"url\":\"https://openbaton.github.io/documentation/pop-openstack/\",\"type\":\"link\",\"title\":\"OpenBaton Documentation\",\"icon\":{\"type\":\"icon\",\"url\":\"https://openbaton.github.io/documentation/pop-openstack/favicon.ico\",\"aspectRatio\":0}}" %}

## Useful resources

* [Open Baton minimal docker setup](https://github.com/openbaton/bootstrap/blob/f9d8ad8a1b25e95baae15da7612f2032bfcff20e/distributions/docker/compose/min_nomysql-compose.yml)
* [Dummy Network Service Record](https://openbaton.github.io/documentation/dummy-NSR/)
* [Dummy NSR docker compose file](https://openbaton.github.io/documentation/compose/dummy-ns.yml)
* [Test Virtual Infrastructure Manager](https://openbaton.github.io/documentation/descriptors/vim-instance/test-vim-instance.json)
* [Test VIM JSON](https://raw.githubusercontent.com/openbaton/docs/master/docs/descriptors/vim-instance/test-vim-instance.json)
* [TOSCA Definition](https://searchcloudcomputing.techtarget.com/definition/TOSCA-Topology-and-Orchestration-Specification-for-Cloud-Applications)


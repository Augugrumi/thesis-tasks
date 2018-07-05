---
description: '''Cuz we don''t like Tacker'
---

# Openbaton

## Preamble

Openbaton is an alternative VIM that can be linked with Openstack. After some developement, we had to change from Tacker to this, so here we are.

## Installations

### On "bare" os

Openbaton installation is pretty straight-forward. Open a terminal and execute the wizard made by the Openbaton team:

```bash
sh <(curl -s http://get.openbaton.org/bootstrap) release
```

Then follow the instruction in the screen and enjoy your Openstack installation!

### Using docker-compose

Sometimes installing Openbaton directly on the OS is not viable, specially if your messing with the it. Every time you break it you have to install everything from scratch, and this is a waste of time. Fortunately, docker come to rescue! From the openbaton team [we've stolen a \(for us\) non-working docker-compose,](https://github.com/openbaton/bootstrap/blob/1b20e2b4ce1ab6e64b110816a7e54f5c0bca48be/distributions/docker/compose/full-compose-with-zabbix.yml) and we've fixed it making it a great tool to deploy openbaton in seconds. From here you can stole our version:

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




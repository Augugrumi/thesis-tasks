---
description: '''Cuz we don''t like Tacker'
---

# Openbaton

## Preamble

Openbaton is an alternative VIM that can be linked with Openstack. After some developement, we had to change from Tacker to this, so here we are.

## Installation

Openbaton installation is pretty straight-forward. Open a terminal and execute the wizard made by the Openbaton team:

```bash
sh <(curl -s http://get.openbaton.org/bootstrap) release
```

Then follow the instruction in the screen and enjoy your Openstack installation!

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




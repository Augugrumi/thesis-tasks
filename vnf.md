---
description: 'Or: finally working on our thesis'
---

# VNF

All our thesis is focus on this, so we hope to do a good job, at least here. What you're gonna find here is a brief explanation of how we built our system to process packets through VNF and get that packets back.

## Harbor: create and destroy VNF in Kubernetes

We're aware that in this kind of environment automation is fundamental, and generally you'd expect that VNF get deployed, scaled and destroyed automatically on-demand, rather than manually accessing your cluster and loading them manually. To accomplish this, we've create a little backend, Harbor, that is capable of creating, destroying, and storing VNF definition.

Harbor, written in Java using the Spark framework, at the time of writing has the following API configuration.

At the end of the day, Harbor ended substituting the Openbaton role: indeed it already covers what Openbaton does, so we see an opportunity to drop the latter and to stick with a component created from us that perfectly fits our needs.

### API definition

{% api-method method="get" host="" path="/vnf/launch/:id" %}
{% api-method-summary %}
Launch a VNF
{% endapi-method-summary %}

{% api-method-description %}
It gives the possibility to launch an already uploaded VNF.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="id" type="string" required=true %}
The VNF id
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{ "result": "ok" }
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="get" host="" path="/vnf/stop/:id" %}
{% api-method-summary %}
Stop a VNF
{% endapi-method-summary %}

{% api-method-description %}
Stop an already running VNF. Note that this destroys all the resources created
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="id" type="string" required=true %}
The VNF id
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{ "result": "ok" }
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="get" host="" path="/vnf/get/:id" %}
{% api-method-summary %}
Get YAML definition of a VNF
{% endapi-method-summary %}

{% api-method-description %}
This API calls return the YAML of a given VNF id.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="id" type="string" required=true %}
The VNF id
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{ "result" : "ok" }
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="post" host="" path="/vnf/create/:id" %}
{% api-method-summary %}
Add a VNF to the catalog
{% endapi-method-summary %}

{% api-method-description %}
Uploads a new VNF to the catalog. Note that the VNF must have an unique id, otherwise the upload will fail. You need to send as the body request a valid Kubernetes YAML.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="id" type="string" required=true %}
The VNF unique id
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{ "result" : "ok" }
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="post" host="" path="/vnf/update/:id" %}
{% api-method-summary %}
Update a VNF
{% endapi-method-summary %}

{% api-method-description %}
Update an existing VNF with a new one. Note that this doesn't modify the resources running in the cluster.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="id" type="string" required=true %}
The VNF id
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{ "result" : "ok" }
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="delete" host="" path="/vnf/delete/:id" %}
{% api-method-summary %}
Delete a VNF from the catalog
{% endapi-method-summary %}

{% api-method-description %}
This method deletes an existing VNF from the catalog. Note that the resources in this case are **not** stopped!
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="id" type="string" required=true %}
The VNF id
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{ "result" : "ok" }
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}



The API json definition can be found here:

```javascript
[
  {
    "name": "/vnf",
    "path": [
      {
        "name": "/launch/:id",
        "type": "get",
        "function": "routes.vnf.VnfLauncherRoute"
      },
      {
        "name": "/stop/:id",
        "type": "get",
        "function": "routes.vnf.VnfStopperRoute"
      },
      {
        "name": "/create/:id",
        "type": "post",
        "function": "routes.vnf.CreateVnfRoute"
      },
      {
        "name": "/delete/:id",
        "type": "delete",
        "function": "routes.vnf.DeleteVnfRoute"
      },
      {
        "name": "/update/:id",
        "type": "post",
        "function": "routes.vnf.UpdateVnfRoute"
      },
      {
        "name": "/get/:id",
        "type": "get",
        "function": "routes.vnf.GetVnfRoute"
      }
    ]
  }
]
```

### Additional notes about implementation

We tried initially to use Kubernetes Java client API, without success, since they're not documented and cryptic. Thus we fell back to a simple, yet horrible, solution, using `kubectl` to deploy YAML on the cluster and paring CLI output.


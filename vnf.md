---
description: 'Or: finally working on our thesis'
---

# VNF

All our thesis is focus on this, so we hope to do a good job, at least here. What you're gonna find here is a brief explanation of how we built our system to process packets through VNF and get that packets back.

## Harbor: create and destroy VNF in Kubernetes

We're aware that in this kind of environment automation is fundamental, and generally you'd expect that VNF get deployed, scaled and destroyed automatically on-demand, rather than manually accessing your cluster and loading them manually. To accomplish this, we've create a little backend, Harbor, that is capable of creating, destroying, and storing VNF definition.

Harbor, written in Java using the Spark framework, at the time of writing has the following API configuration:

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

We tried initially to use Kubernetes Java client API, without success, since they're not documented and cryptic. Thus we fell back to a simple, yet horrible, solution, using `kubectl` to deploy YAML on the cluster and paring CLI output.


---
description: How VNFs speack each other
---

# Astaire

The communication among VNFs is implemented using UDP protocol and in order to do so we developed [astaire](https://github.com/Augugrumi/astaire). Astaire is basically a software that works both as UDP server and UDP client. 

![How Astaire works](.gitbook/assets/astaire-algorithm.png)

In that new version of our implementation each VNF has the capability of receiving UDP packets and sends them to the next element of the chain. In order to make everything works 2 elements at the 2 ends of the chain are needed: the head of the has the aim to receive packets from a TUN tunnel, contact the classifier in order to define the chain and then to send the original packet, wrapped into a UDP packet, to the first "real" VNF. The _tail,_ likewise, has to unwrap the packet modified by the chain of VNF, sending it to the original receiver.

![](.gitbook/assets/astaire.png)

### Deploy

Astaire can be run on machine, but for the sake of this project we developed a Docker container in order to deploy the chain of VNFs on Kubernetes. The Dockerfile is available [here](https://github.com/Augugrumi/astaire/blob/master/Dockerfile), and it is based on Alpine Linux, to make the Docker image as slim as possible. 

### The chain

In order to deploy the chain of VNFs we take advantage of Kubernetes and of the Astaire's container. Basically as in the one of the previous solutions that we proposed we expose each VNF with a service: doing so we have the possibility to easily scale the VNFs and we can work on a higher abstraction layer. As a matter of fact that we do not use physical IP addresses of machine and/or pods but we can use Service's names, making all the system more flexible.

This architecture as has a drawback: Kubernetes has a centralized DNS systems so, for each request, the pods that works as DNS must be queried. This can slow down the performance of the whole system as it became the bottleneck of the network.

To outflank the issue we used [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html). It is a lightweight DNS forwarded that allows to cache requests made to real recursive DNS. We deploy it as a Kubernetes daemonset and that make possible to create a pod for each machine: each VNF, so, can query the dnsmasq pod to have a lower latency name resolution and to avoid to flood the node with the DNS.

{% code-tabs %}
{% code-tabs-item title="daemonsetdns.yaml" %}
```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: dnsmasq
  namespace: kube-system
  labels:
    k8s-app: dnsmasq-cache
spec:
  template:
    metadata:
      labels:
        name: dnsmasq
    spec:
      hostNetwork: true
      nodeSelector:
        beta.kubernetes.io/arch: amd64
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
      containers:
        - name: dnsmasq
          image: andyshinn/dnsmasq:latest
          args:
            - "-S"
            - "/cluster.local/10.96.0.10 /kubernetes.default/10.96.0.10"
          ports:
            - containerPort: 53
              hostPort: 53
              protocol: UDP
            - containerPort: 53
              hostPort: 53
              protocol: TCP
          securityContext:
            capabilities:
              add:
                - NET_ADMIN
      terminationGracePeriodSeconds: 30
```
{% endcode-tabs-item %}
{% endcode-tabs %}

dnsmaqs containers have to be run with `NET_ADMIN` capabilities, making possible to access the `/etc/resolv.conf` file. 

To make this enhancement effective, even the YAML file that describes the chain to be deployed have to take in account the local dnsmasq name resolution. To do so, ideally, containers of the chain must be added with the following lines:

```yaml
dnsConfig:
  nameservers:
    - <hostIP>
```

Where `hostIP` is the IP of the machine on which the pod is running. Unless for args that can be passed to a container, it is not possible to use [Kubernetes downward API](https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/#the-downward-api) to access `status.hostIP` variable. 

The workaround that we implement is to add to the containers environment a variable set to the host IP launch Astaire inside the container using a bash script:

```bash
#!/bin/sh

set -e

if [ -n "$KUBERNETES_PORT" ] && [ -n "$DNS_IP" ]
then
    tmpfile=$(mktemp)
    (echo "nameserver $DNS_IP" && tac /etc/resolv.conf) > $tmpfile
    cat $tmpfile > /etc/resolv.conf
    rm $tmpfile
fi

astaire $@
```

Where `KUBERNETES_PORT`  allows us to know if the container is running on Kubernetes and `DNS_IP` is the host IP that we retrieve using the Kubernetes downward API passing the host IP as argument to the Astaire container. `DNS_IP` is set adding to the YAML file of VNFs the lines:

```bash
env:
  - name: DNS_IP
    valueFrom:
      fieldRef:
        fieldPath: status.hostIP
```




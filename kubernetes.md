---
description: A heaven for your containers
---

# Kubernetes

## Persistent Storage

### GlusterFS

To enable persistent storage in Kubernetes with Glusterfs you need at least three nodes. As we've already written above, we deployed our Kubernetes cluster using CentOS 7.

Since we're running our cluster on a Openstack environment, we attached one volume per VM. If you're in our shoes, just create some empty volumes and attach them to nodes you want to configure with glusterfs.

First of all, you need to install `gluster-server` on all the nodes that will join the storage and `gluster-fuse` to provide `mount.glusterfs` as an option to mount Gluster file systems. Open a terminal and type:

```bash
sudo yum install -y centos-release-gluster gluster-server gluster-fuse
```

At this point, enable the `glusterd` daemon to start at every system start:

```bash
sudo systemctl start glusterd && sudo systemctl enable glusterd
```

Our next step is to load different kenel modules, using `modprobe`:

```bash
sudo sh -c 'modprobe dm_snapshot && modprobe dm_mirror && modprobe dm_thin_pool'
```

To make this change permanent, add the lines above to your `/etc/rc.local`.

[According the gluster-kubernetes setup guide](https://github.com/gluster/gluster-kubernetes/blob/master/docs/setup-guide.md), you should also check that the following ports are reachable:

* 2222
* 24007
* 24008
* 49152-49251

On top of that, you'll need to grant access to your root account while installing all the stuff we need. In CentOS, if like us you're using the CloudImg image, you need to access every machine and type:

```bash
sudo su # gain superpowers
cd /root/
nano .ssh/authorized_keys
# edit the first line in the follwing way: from the sub-string starting
# as "ssh-rsa" untill the end put the content in a new line, while comment
# the whole line
nano /etc/ssh/sshd_config # un-comment "PermitRootLogin: yes"
systemctl restart sshd
```

At this point, you need to clone kubernetes-glusterfs, the repository that contains a magic script that will figure out all the necessary stuff \(e.g. heketi\), `gk-deploy`.

```bash
git clone https://github.com/gluster/gluster-kubernetes.git
cd gluster-kubernetes/deploy
```

We're reaching our goal, hold on! Now we need to define our tolopoly. To do so, create a `tolopogy.json` from the sample \(`topology.json.sample`\) and edit to fit your neccessities. Here, we report our:

{% code-tabs %}
{% code-tabs-item title="topology.json" %}
```javascript
{
 "clusters": [
   {
     "nodes": [
       {
         "node": {
           "hostnames": {
             "manage": [
               "192.168.29.26"
             ],
             "storage": [
               "192.168.29.26"
             ]
           },
           "zone": 1
         },
         "devices": [
           "/dev/vdb"
         ]
       },
       {
         "node": {
           "hostnames": {
             "manage": [
               "192.168.29.19"
             ],
             "storage": [
               "192.168.29.19"
             ]
           },
           "zone": 1
         },
         "devices": [
           "/dev/vdb"
         ]
       },
       {
         "node": {
           "hostnames": {
             "manage": [
               "192.168.29.22"
             ],
             "storage": [
               "192.168.29.22"
             ]
           },
           "zone": 1
         },
         "devices": [
           "/dev/vdb"
         ]
       }
     ]
   }
 ]
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

As you can see, in "manage" we have put the IP address of our machine, even if we already defined their addresses in `/etc/hosts`. 

Ta-daan! 🎉 Now we're ready to deploy our Gluster cluster. From the master, type in a terminal the following command:

```bash
# assuming you still are in ~/blabla/gluster-kubernetes/deploy/
./gk-deploy -s <path/to/your/private/key> --ssh-user root --ssh-port 22 topology.json -y
```

At the end, if everything goes correctly, you should see a `StorageClass` template. Copy it deploy to your Kubernetes through `kubectl`.

```bash
kubectl create -f glusterstorageclass.yaml
```

Optionally, you can set it as default `StorageClass` with the following command:

```bash
kubectl patch -f glusterstorageclass.yaml \
-p'{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```




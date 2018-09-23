---
description: New chaining functionality
---

# Connection with the chain

After initial proposals, we refined our implementation adding 2 improvement:

* We implemented the connection of sender and receiver of the packet with the chain with a TUN tunnel
* We implement send and receive on the core of the VNF using UDP instead of TCP

## TUN/TAP

TUN and TAP are virtual kernel interfaces. These differs from the normal network interfaces because they are completely software and no physical components are required. 

TUN \(network TUNnel\) operates with layer-3 packets \(as IP packets\), TAP instead \(network tap\) works on layer 2, with Ethernet packets. When used with user-space programs TUN/TAP devices injects packets to the operating-systems network stack: this emulate the reception from an external source. 

In order to manage the communication on a tun/tap interface a program gets a file descriptor to which read and write data in order to receive or send them. To the kernel, it would look like to receive packets "from the wire".

These interfaces can be both **transient** and **persistent**. Transient means that the interface is created, used and destroyed by the same program, persistent means that are used some utility to create them and then programs can attach to it.

Once create the interfaces can be used as the usual "hardware" network interfaces. 

TUN e TAP interfaces are used for VPNs, virtual machine networking, to connect physical machines to network emulators and NATs. 

![TUN interface representation](.gitbook/assets/tun.png)

### Network tunnel

Network tunneling protocol is a communication protocol that make possible exchange of data in secure ways across public networks and to run a protocol that is not supported by a certain network thanks to the encapsulation. Basically, the tunneling protocol works by using the payload of a packet to carry the packets that actually provide the service. 

### Network TAP

Network Terminal Access Point is a external monitoring device that mirrors the traffic that pass through two nodes, instead a TAP is the device in the network that actually monitor the traffic. 

## TunConnector

The first change we performed is to create a TUN tunnel between VNFs. This allow us to better manipulate packages: encapsulating the message send from the sender we can exchange a Layer-3 packet among the VNFs as payload of a UDP packet. In that way we can maintain most of the original information, so we can send the packet to the right destination.

In order to accomplish that, we create [TunConnector](https://github.com/Augugrumi/TunConnector): this is a naive implementation of a software that create a virtual tun interface on the host machine and than allows the connection to another machine or waits for connection. This software was developed following this [tutorial](https://backreference.org/2010/03/26/tuntap-interface-tutorial/) and starting from [simpletun](https://github.com/gregnietsky/simpletun) code.

TunConnector requires to specify the port on the host used to create the TUN tunnel and the IP that will be used in it to reach the machine.

### How it works

TunConnector can work as a client, server or both modality \(for a certain link the host will be a client, for another the server\). The first operation that it performs in each configuration is to create a TUN device using an [OpenVPN](https://openvpn.net/) utility:

```bash
# openvpn --mktun --dev tun3
# ip link set tun3 up
# ip addr add 192.168.0.2/24 dev tun3
```

In order to perform that operation TunConnector must run as root. First command create an interface, in this case called `tun3`. The second one turns up the newly created interface. Last one set the IP address \(in the example above 192.168.0.2\) that will be used in the tunnel. This is a naive way to create a persistent interface: in fact it is possible to do it using C++, but for the sake of simplicity and because of the controlled environment in which we work, that is the best choice.

After both in client and in server mode it allocates 2 socket, retrieving the correspondent file descriptors: one is used for the communication inside the tunnel, the other to communicate with the "usual" network. In case of server mode after it binds a socket to a specific port in order to listen for requests of connections. 

Once a server receive a request of connection from a certain client, the tunnel will be created. This tunnel consist of a infinite loop in which data for the tunnel are redirected to the right file descriptor from the network and vice versa.

### Example

Usage:

```bash
$ sudo ./TunConnector [OPTIONS]
```

* `--serverinterface -p X` To set the name if the interface used by the server component
* `--clientinterface -n X` To set the name if the interface used by the client component
* `--serverip -s X` To set the IP on the server interface
* `--clientip -c X` To set the IP on the client interface
* `--inport -i X` To set the port used by the server component. Default `55556`
* `--remoteip -r X` To set the ip of the remote server to connect to
* `--mode -m [client|server|both]` To set run mode. Default `both`
* `--help -h` Show the help



with the following options

{% tabs %}
{% tab title="Server" %}
`$ sudo ./TunConnector -p tun0 -s 10.0.0.2 -i 55555 -m server`
{% endtab %}

{% tab title="Client" %}
`$ sudo ./TunConnector -n tun0 -c 10.0.0.3 -o 55555 -i 192.168.6.9 -m client`
{% endtab %}
{% endtabs %}

In the example above we suppose to create a server on a host with IP 192.168.6.9 \(the IP is required from the client\), that accept requests on port 55555. The interface will be named tun0 and inside the tunnel the machine will have the address 10.0.0.2.

In an another machine the client run in a mirrored way the program, specifying the server port and IP address.

Once the tunnel is up, you can check that the connection is working for example pinging the two ends of the tunnel with the IP address used inside the tunnel.  


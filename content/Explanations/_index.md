---
Title: Explanations
---

## Find explanations and definitions for common terms & concepts related to self-hosting

### Topics

Explainations:

- [Reverse Proxies]({{< relref "#reverse-proxies" >}} "Reverse Proxies" )
- [Port Forwarding]({{< relref "#port-forwarding" >}} "Port Forwarding" )
- [Containers]({{< relref "#containers" >}} "Containers" )
- [Virtualization]({{< relref "#virtualization" >}} "Virtualization" )
- [Virtual Private Networks]({{< relref "#virtual-private-networks" >}} "Virtual Private Networks" )

#### Reverse Proxies

Reverse proxies are daemons that accept connections and then connect to another service based on port or host to facilitate the request. They act as a middle man instead of a traffic redirector.

Typical use cases for reverse proxies are to provide a unified frontend for multiple backends or hosts. Another common use is for high-availability to provide failover or distribute load between multiple backends serving the same content.

Examples of popular software capable of performing as a reverse proxy are: squid, haproxy, apache, nginx, and caddy.

[Top]({{< relref "#topics" >}} "To the top" )

#### Port forwarding

Port forwarding is the function of inspecting traffic on an incoming port and redirecting it to another port or host with minimal modification. Primary purposes of this is to forward traffic to a service behind a firewall/router.

Common for hosting game servers from home when running dedicated servers before developers moved to match-making. Another use for this is to open ports for bittorrent so that you can share your vast and innumerable collection of Linux ISOs.

The difference between port forwarding and a reverse proxy is that the reverse proxy will accept, process, and establish a new connection to the backend service to fulfil the request.

Port forwarding inspects and alters packet headers before it is routed to its new destination. The connection is otherwise untouched.

Port forwarding is a function of your firewall. Commonly at the router or other network gateway.

Linux has one firewall called **iptables** with many frontends or management packages available for it. BSD based firewalls are **PF**, **IPFW**, and **IPFILTER**. The Windows firewall consists of a scarecrow holding a sign saying: "plz no tresspass".

[Top]({{< relref "#topics" >}} "To the top" )

#### Containers

Containers are software envelopes to isolate a piece or bundle of software and their dependencies. Containers come in may forms.

A container could contain a PHP based forum with an AMP stack (Apache, Maria DB, PHP ) as dependencies.

This is useful if you want an easy way to deploy software without configuring dependent software/libraries manually.

Containers can also resolve software conflicts when running multiple services which depend on different versions of the same software/libraries.

Popular containers are Linux Containers ( LXC ), jails ( BSD/Unix ), Kubernetes, and Docker.

[Top]({{< relref "#topics" >}} "To the top" )

#### Virtualization

Virtualization is a lower level form of containerization. There a many forms of virtualization that provide different sets of features/tradeoffs.

In practice, it often virtualizes whole, or major parts of an operating system.

##### Full virtualization

Full virtualization is generally the containerization of a full, unmodified operating system with virtualizaed hardware. The virtualized OS is not host-aware.

Fully virtualized guests require more overhead than paravirtualized guests. This can be mitigated with hardware support ( Intel VT, AMD SVM ) for virtualization instructions.

Examples of this are Hyper-V, Xen, KVM/Qemu, VMware ESX(i)

##### Para-Virtualization

Para-virtualization is the practice of running a modified kernel/OS where privileged instructions are sent through an API shared with the host. It does not require the virtualization of hardware, but it does require an operating system that is modified to be used with the specific API used by your chosen virtualization method. This can be in the form of source code modifications or specialized device drivers.

Microsoft Windows cannot be paravirtualized.

Examples: Xen, Oracle VM, OpenVZ.

[Top]({{< relref "#topics" >}} "To the top" )

#### Virtual Private Networks

Virtual Private Networks ( VPNs ) are a way of networking individual machines together in software regardless of their physical or network proximity.

A typical use case is for networking corporate locations together to share network resources such as file shares, intranet webservers, on-premesis services, etc.

Another use for a VPN is to tunnel traffic destined for a public service through to another endpoint, usually to bypass geo-location restrictions or state-imposed censorship of the internet.

Some use VPNs to keep services behind a restrictive ISP or firewall accessible outside of said network.

[Top]({{< relref "#topics" >}} "To the top" )

### ECMP

Enabling multiple paths simply requires the configuration of ECMP.
SR Linux devices allow for up to 128 ECMP paths.
For an eBGP underlay, we configure ECMP under the default network instance.
See the configuration snippet below for BGP ECMP.


```
--{ runnning }--[  ]--
A:srl1# info network-instance default
    network-instance default {
        protocols {
            bgp {
                ipv4-unicast {
                    multipath {
                        max-paths-level-1 16
                        max-paths-level-2 16
                    }
                }   
            }
        }
    }
```

The levels in the above configuration are described as follows:

* `max-paths-level-1` – next-hop-group is bound directly to a BGP IPv4 prefix
* `max-paths-level-2` – next-hop-group is bound to the IPv4 or IPv6 route used to resolve a BGP next-hop (for RFC 5599, advertising/receiving IPv4 routes over IPv6 next hops)

In SR Linux platforms, the ECMP hash is calculated from the following criteria:

* hash-seed: value -> 0-65535, configure different values on each router to avoid traffic polarization effects
* protocol: IPv4 header protocol value or the IPv6 next-header value in the last extension header
* source-ip: the source IPv4/IPv6 address
* dest-ip: the destination IPv4/IPv6 address
* src-port: the source TCP/UDP port number
* dest-port: the destination TCP/UDP port number

We can use the table below to determine hash algorithm calculation for each packet type:

Packet Type                  | Hash Calculation
---------------------------- | --------------------------------------------------------------------------------
IPv4-TCP non-fragment packet | hash-seed, source-ip, dest-ip, protocol, TCP src-port, TCP dst-port
IPv6-TCP non-fragment packet | hash-seed, source-ip, dest-ip, flow-label, protocol, TCP src-port, TCP dst-port
IPv4-UDP non-fragment packet | hash-seed, source-ip, dest-ip, protocol, UDP src-port, UDP dst-port
IPv6-UDP non-fragment packet | hash-seed, source-ip, dest-ip, flow-label, protocol, UDP src-port, UDP dst-port

### eBGP Optimizations - Tuning eBGP for a DC Fabric

As BGP is commonly used for WAN connections, the default timers for path updates and advertisements need to be tuned for a richly connected data center environment to enable faster convergence and updates.
   
#### Timers and Convergence Optimization

Since a single data center will likely consist of hundreds or even thousands of nodes, it is important to simply the configuration and generalize the best timer values to fit most situations.
As previously mentioned, the default timers are typically sufficient for a WAN centric design often utilized by service providers.
In the data center, it is important to both maintain stability and match the convergence speed typically seen in link state protocols.
Modifying a few main timers will help to achieve this goal.

#### MRAI (Minimum Router Advertisement Interval)

First, let us look at the `advertisement interval`, which determines how much time must elapse between an advertisement or withdrawal of routes to a BGP peer.
The default value of 30s for eBGP peers is not acceptable in a data center environment, so the recommendation is to change this value to 1s.
Rather than configuring this for each peer, it can be applied to a peer group and inherited by each peer to which it is applied.

```
--{ running }--[  ]--
A:srl1# info network-instance default
    network-instance default {
        protocols {
            bgp {
                group cloud {
                    timers {
                        minimum-advertisement-interval 1
                    }
                }
            }
        }
    }
```

#### Keepalive and hold timers

By default, the `keepalive-interval` and `hold-time` values are 30s and 90s respectively.
This means that a BGP peer will send a keepalive message for a session every 30 seconds.
If the adjacent peer does not see a keepalive message for 90 seconds, the session will be torn down.
In the data center, these values seem like a lifetime.
To optimize convergence while also maintaining stability, it is recommend to change the keepalive value to 3 seconds and the hold-down timer to 9 seconds.
See the sample configuration below:

```
--{ * candidate shared default }--[  ]--
A:srl1# info network-instance default
    network-instance default {
        protocols {
            bgp {
                group cloud {
                    timers {
                        hold-time 9
                        keepalive-interval 3
                        minimum-advertisement-interval 1
                    }
                }
            }
        }
    }
```

The above timers come into play when a BGP peer becomes unreachable.
Using an additional protocol such as BFD will allow even faster convergence.

### BFD (Bidirectional Forwarding Detection)

BFD is a lightweight protocol that can be enabled on a point-to-point link between routers to detect link failures.
It is designed to leverage hardware forwarding to enable sub-second failure detection.
Once a failure is detected, the link is immediately marked down and upper layer protocols react accordingly (withdraw route, recalculate ECMP, etc.).
Because BFD is a protocol unto itself, it can be used in combination with many different routing protocols to assist with faster convergence.
In the case of BGP, SR Linux supports BFD for both IPv4 and IPv6 peering.

There are two aspects of the BFD configuration.
BFD must first be configured for a given subinterface and then associated with a routing protocol.
Within BGP, BFD can be configured at the global, group or neighbor level.

In this example, BFD is configured on subinterface `ethernet-1/2.1`.  Timer values are specified in microseconds. The minimum value is 10,000 microseconds (0.01s).

```
--{ running }--[ ]--
# info bfd
    bfd {
        subinterface ethernet-1/2.1 {
            admin-state enable
            desired-minimum-transmit-interval 250000
            required-minimum-receive 250000
            detection-multiplier 3
        }
    }
```

Next, BFD is associated with BGP `cloud` peer group.

```
--{ running }--[ ]--
# info network-instance default protocols bgp
    network-instance default {
        protocols {
            bgp {
                group cloud {
                    failure-detection {
                        enable-bfd true
                    }
                }
            }
        }
    }
```

###	Maintenance Mode

Maintenance mode is a feature that allows you to take a network element out of service so that maintenance actions can be performed such as upgrading the software image.
This feature can be applied to other applications like BGP as well.
For example, a maintenance group can consist of one or more BGP neighbors and/or peer groups belonging to one or more network instances.

Using Maintenance Mode in a Clos data center fabric will allow network operators to take advantage of the multipath capabilities of said architecture while minimizing downtime.
If we reference **Figure 3-4**, we can apply maintenance mode one of the spines in AS 64550.  

<figure>
  <img src="/_images/fig-03-04.png" width="600" />
  <figcaption>Figure 3-4</figcaption>
</figure>

In SR Linux systems, constructs called maintenance groups are used to associate specific objects, such as entire network instances or VRFs, BGP peer groups and neighbors with a maintenance profile.
We can create a maintenance profile to specify certain policy changes to a BGP peer group or neighbor.
An example policy will prepend additional AS’s in the path to gracefully force traffic to the redundancy spine in AS 64550.
When maintenance mode is enabled under the group, the policy is put into effect to cause the desired action. 

Refer to the [Maintenance Mode](../appendix/maintenance.md) section of the Appendix for more examples for configuring maintenance mode.

###	Network Addressing

When designing the network fabric addressing scheme, the most common practice is use private addressing as described in RFC1918 for IPv4 networks and add an additional layer of load balancers and/or border routers to provide the NAT function for communication to external networks.
Public IPv4 address space is typically reserved for WAN and Internet connections since that address space is becoming more and more exhausted.

IPv6 alleviates the address exhaustion due to the sheer amount of addresses available.
It is recommended to manually allocate appropriate address blocks per RFC4193 using a RFC4086 compliant random number generator to improve security.
RFC4193 addresses are intended for local communications and are not expected to routable within the public domain.
RFC4086 helps to ensure the allocated addresses are resistant to scanning attacks.
Avoid manual addressing formats that either mirror IPv4 based addresses or incorporates words.

This flexibility with IPv6 addressing, comes at the cost of requiring some type of interopability with IPv4 networks.
IPv4 addressing is still the default in today’s networks and some applications may only support IPv4, so a transition is inevitable to reap many of the benefits of IPv6.

#### Addressing Schemes

If we consider the network pictured in **Figure 3-3** and use IPv4 addressing for the fabric links the following addressing scheme could be used.
The spine nodes and directly connected leaf nodes associated with AS 64550 are "POD1" and the respective nodes associated with AS 64560 are "POD2".
Assuming a contiguous address block is associated with each POD and assuming IPv4 addresses: POD1 links and loopbacks will be part of subnet block 192.168.10.0/28, and POD2 links and loopbacks will part of 192.168.10.16/28.
Each block contains enough addresses to carve out /31 subnets and /32 loopbacks required for the 4 node POD.
If possible, end hosts connected to the leaf nodes should be assigned addresses out of a subnet that is local to a given leaf.
In general, it is best to avoid subnets that span multiple racks.
If there is a need for stretching subnets across racks then consider using an overlay virtual network such as EVPN/VXLAN.
(Network Virtualization and Overlay networks will be covered in a future section of this data center blueprint.)

#### Advertising IPv4 Routes Using IPv6 Next Hops: RFC5549

Some data centers are migrating away from a purely IPv4 or dual stack environment to a fully IPv6 fabric.
For small to medium size businesses, IPv4 is sufficient to support the number of devices in the network; however, larger enterprises and webscale companies may find themselves running out of IPv4 private address space, hence the need the for IPv6 implementations.
In any case, the transition to IPv6 does not occur overnight, so a migration plan is required. 

Let’s consider two use cases:

1. The network operator has deployed a fully IPv6 data center fabric but must still interop with IPv4 only networks during the transition period.
2. The data center fabric is fully IPv4 but must interop with an IPv6 network.

In either case, RFC5549 can solve the need for transition by providing a standard to advertise IPv4 routes using IPv6 next hops.
Each interface between BGP peers only needs one or more IPv6 interfaces defined. 

<figure>
  <img src="/_images/fig-03-05.png" width="600" />
  <figcaption>Figure 3-5</figcaption>
</figure>

In order to route and forward IPv4 packets over an IPv6 network, the data center node most support the following:

1. The ability to advertise a BGP route for an IPv4 NLRI (Network Layer Reachability Information) with an IPv6 BGP next-hop address.
2. The ability to receive a BGP route for an IPv4 NLRI with an IPv6 BGP next-hop address

To enable this behavior, the BGP process must be configured to both advertise and receive IPv4 packets on the IPv6 interfaces, otherwise the packets will be discarded.
See the configuration snippet below:

```
--{ running }--[  ]--
A:srl1# info network-instance default
    network-instance default {
        protocols {
            bgp {
                group cloud {
                    ipv4-unicast {
                        advertise-ipv6-next-hops true
                        receive-ipv6-next-hops true
                    }
                }
            }
        }
    }
```  

### Route Summarization

Route summarization is one of the techniques that allow routing to scale to support very large networks.
For Clos-based networks, however, it is important that route summarization is used carefully.
Generally, Clos networks will only summarize routes at the ToR/leaf, especially when a subnet is contrained to a single rack.

When deploying many hundreds or thousands of fabric switches, it may be beneficial to summarize at the super spine layer between the "PODs".
Recall **Figure 3-3** and the addressing scheme mentioned above:

> The spine nodes and directly connected leaf nodes associated with AS 64550 are "POD1" and the respective nodes associated with AS 64560 are "POD2".
> Assuming a contiguous address block is associated with each POD and assuming IPv4 addresses: POD1 links and loopbacks will be part of subnet block 192.168.10.0/28, and POD2 links and loopbacks will part of 192.168.10.16/28.
> Each block contains enough addresses to carve out /31 subnets and /32 loopbacks required for the 4 node POD.

The super spines that interconnect POD1 and POD2 can summarize the routes for each POD such that any node in POD1 that needs to reach any node in POD2 will only require any one of the superspine nodes to advertise subnet 192.168.10.16/28.
This summarization design will ensure that route tables stay relatively small for underlay routes, and that the spines remain as simple as possible.

Routing and route summarization in the case of network virtualization (EVPN/VXLAN overlays) will be considered in a future revision of this document.

### Route Policy

The route policy for the fabric should kept as simple as possible to allow the policy configuration to remain repeatable throughout the data center fabric.
We can implement simple export policies that export all BGP and local routes, and import policies that accept all BGP routes from adjacent BGP peers.

Sample export policy:

```
--{ running }--[  ]--
A:srl1# info routing-policy
    routing-policy {
        policy export {
            default-action {
                reject {
                }
            }
            statement 10 {
                match {
                    protocol bgp
                }
                action {
                    accept {
                    }
                }
            }
            statement 20 {
                match {
                    protocol local
                }
                action {
                    accept {
                    }
                }
            }
        }
    }
```

Sample Import policy: 

```
--{ running }--[  ]--
A:srl1# info routing-policy
    routing-policy {
        policy import {
            default-action {
                reject {
                }
            }
            statement 10 {
                match {
                    protocol bgp
                }
                action {
                    accept {
                    }
                }
            }
        }
    }
```

The policies are easily repeatable and can be applied to all data center fabric nodes in the same way, as they do not contain specific IP addresses or other node specific items.

The super spine layer will have slightly more complicated policies to allow for route summarization.
Unique policies may also be required to tie into the aggregation nodes, those policies is beyond the scope of this document.
The fortunate part here is that a specific super spine policy can repeated across all super spine nodes, so there is no need to create unique policy for each device.

###	Dynamic BGP Peering

Dynamic BGP peering is a feature that allows a network operator to simplify BGP configuration for a large number of peers based on specific match criteria.
In SRL devices, Dynamic BGP peering is supported for both IPv4 and IPv6 peers.
Match criteria can include either IP address blocks, ASN ranges, or both.
This feature allows a repeatable configuration across devices, and most useful for automatically connecting downstream devices for spine and super spine devices in a 5-stage Clos architecture.

See the configuration snippet below for example of matching on IP and ASN criteria:

```
--{ running }--[  ]--
A:srl1# info network-instance default
    network-instance default {
        protocols {
            bgp {
                dynamic-neighbors {
                    accept {
                        match 192.168.10.0/24 {
                            peer-group spine
                            allowed-peer-as [
                                64550
                            ]
                        }
                    }
                }
            }
        }
    }
```

The above configuration could be applied to the super spine for example to automatically peer with all spine switches in specific pod.

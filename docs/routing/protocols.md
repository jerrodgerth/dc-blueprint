# Fabric Routing

## Protocol Selection

Most organizations already use several different routing protocols so what protocol is best for a data center fabric?
This topic has been covered by many different network operators and vendors over the last few years.
[Routing Design for Large Scale Data Centers: BGP is a Better IGP](https://archive.nanog.org/meetings/nanog55/presentations/Monday/Lapukhov.pdf) and [RFC 7938 - Use of BGP for Routing in Large-Scale Data Centers](https://tools.ietf.org/html/rfc7938)
The concensus choice for a data center routing protocol is to use BGP and specifically eBGP.
(Certainly other protocols, such as OSPF or IS-IS, could be used and are supported by SR Linux but the focus of the document will be on BGP.)
BGP lends itself well to the data center use case for several key reasons:

1. Multi-hop Multipathing
2. Redundancy and Resiliency
3. Scalability
4. Multiprotocol Support
5. Simplicity
6. Operations

Let's look at each of these in more detail.

### Multi-hop Multipathing

Forwarding traffic over many available paths is a fundamental requirement for a Clos-based fabric for both throughput and resiliency.
Routing protocols implement Equal-Cost Multipath (ECMP) to forward traffic over many paths (often 64 or more) assuming all paths are "equal."
Examples of how ECMP is configured in SR Linux will be covered later in this document.

Unlike link state protocols, BGP supports ECMP routes that consist of multiple hops. In case of eBGP, this path is tracked in the AS Path attribute, 
and multi-hop loops are prevented through the BGP route acceptance algorithms.

### Redundancy and Resiliency

The data center fabric must have the ability to minimize the effect of losing a single link or node and provide fast re-convergence.
Consider a simple spine-leaf data center fabric in **Figure 3-1**.

<figure>
  <img src="/_images/fig-03-01.png" width="600" />
  <figcaption>Figure 3-1</figcaption>
</figure>

Each leaf in the design has 4 uplinks, with each uplink connecting to a different spine in the fabric.
With multipathing enabled each uplink link is utilized under normal condititions.
Losing a single spine switch will result in a 25% loss in traffic forwarding capability.
The same principles can be applied to scheduled maintenance.
For example, each spine in Pod1 can be taken down systematically for an upgrade without interrupting service for the entire Pod. 
Depending on the fabric design and tolerance for loss, operators can rely on fast failover mechanisms to re-route traffic with minimal traffic loss.
Alternatively, by updating the BGP policies on the node in maintenance, traffic can be gracefully diverted without loss of in-flight packets.
Once the ECMP route through the node in maintenance has been withdrawn, traffic will be redistributed by ECMP over the remaining links between leaf and spine.

### Scalability

Clos-based networks can scale to support hundreds or even thousands of racks using mutliple tiers (or stages) as discussed in the previous chapter.
In large networks, it is critical that the chosen routing protocol can also scale to hundreds or thousands of routers.
Path-vector routing protocols like BGP are primarily concerned about maintaining state with their peer routes as opposed to link-state protocols such as OSPF which require understanding of the entire network.
This behavior provides BGP an advantage in large networks.
As mentioned before, OSPF or IS-IS could be used to build data center networks but introduce extra complexity for very large scale deployments.
(Detailing of these differences is beyond the scope of this project, but please *reach out* if you would like to discuss further.)

Using eBGP specifically, along with well-designed use of private AS numbering, makes BGP particularly well suited to large scale deployments.
These considerations will be discussed in more detail in the [BGP Configuration](bgp-config.md) section.

### Multiprotocol Support

Modern data center networks often require both IPv4 and IPv6 support and perhaps even EVPN or MPLS for virtual networks.
Multiprotocol support is a key strength of BGP.
IPv4, IPv6, EVPN and more can be supported in a single peering session through a feature known as a *network address family*.
RFC5549 leverages these capabilities of BGP and outlines the ability to support "IPv4 over IPv6".
This RFC and its applicability will be discussed later in this guide.

### Simplicity

Simplicity is an important part of operating networks at scale.
If a single protocol can be used instead of two or even three that has value.
Modern data center networks that host applications running on Kubernetes for example will often recommend running BGP directly on servers and routing all the way to the end host.
This brings some benefits but it also requires administration of a routing protocol on the end host.
Being able to use the same protocol (BGP in this case) makes interoperability, configuration and operations much easier.

### Operations

Once understood, BGP is a simple and straightforward protocol.
It presents a clear and easy to understand routing table.
Because it is a path vector protocol, when combined with well designed AS numbering it becomes quite easy to troubleshoot by looking at the `AS_PATH` attribute.
Lastly, because BGP is an open standard there are many good implementations that are able to interoperate and provide choice for end users.

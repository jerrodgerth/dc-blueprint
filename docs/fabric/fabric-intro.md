# Fabric Design Considerations

As we look to implement best practices for building data center networks, it is important to realize there is a not a "one size fits all" answer.
While there are certainly *some* best practices to follow, each network architect must understand their own environment and implement accordingly.
That said, there are clear benefits to moving away from legacy network designs that were designed primarily using large Layer 2 domains and supporting North-to-South (human-to-machine) traffic patterns.
The rise of micro-services applications and East-to-West (machine-to-machine) traffic patterns is forcing the network to be re-designed to meet these new demands.

## Background

Legacy data center designs were generally composed of three-tiers - access, aggregation and core (see **Figure 2-1**).
Servers (usually bare metal or early virtualized hosts) connected to the access layer.
The aggregation and core layers were often large chassis-based platforms.
Layer 2 and Layer 3 traffic was segregated at the aggregation layer.

<figure>
  <img src="/_images/fig-02-01.png" width="600" />
  <figcaption>Figure 2-1</figcaption>
</figure>

This design has several inherent disadvantages:

1. Scalability of the aggregation and core layers
2. Sub-optimal and non-deterministic traffic paths
3. Inter-subnet traffic tromboning via aggregation and/or core
4. The use of xSTP as a Layer 2 anti-loop mechanism blocked links and left them unused

Evolutionary techniques such as MC-LAG helped to reduce the need for spanning-tree and provide server access redundancy (see **Figure 2-2**) but the other disadvantages remain.
Some vendors and standards bodies pursued solutions such as TRILL and SPB but these efforts did not gain significant market adoption.
In the end, as in many other instances, rather than creating a new protocol the market fell back on IP routing and standard routing protocols such as BGP.

<figure>
  <img src="/_images/fig-02-02.png" width="600" />
  <figcaption>Figure 2-2</figcaption>
</figure>

For several years, the largest network providers have been building scale-out networks using small, fixed-configuration switches.
These switches are arranged according to [Clos](https://en.wikipedia.org/wiki/Clos_network) design principles.
Clos networks are named after one of the scientists who popularized this method, Charles Clos, and it originally defined an efficient way of building high-capacity, scale-out circuit switching networks.

Modern data centers can use these same principles to build Layer 3, routed networks (see **Figure 2-3**).
Networks (or Fabrics) designed in this way, overcome all of the limitations listed above with one drawback - Layer 2 domains are now confined to a individual (or pair) of access (also called Leaf or Top of Rack (ToR)) switches.
To support Layer 2 domains beyond this limit, network virtualization technologies such as VXLAN and EVPN can be used.
This is an important topic and will be discussed in a later section.

<figure>
  <img src="/_images/fig-02-03.png" width="600" />
  <figcaption>Figure 2-3</figcaption>
</figure>

A simple Clos network is comprised of leaf switches and spine switches (See **Figure 2-3**).
The leaf (or ToR) switches connect to the end devices/hosts and are connected together via the spine switches.
Every leaf switch is connected to every spine and thereby is at most one "hop" away from any other leaf in the network.
These leaf/spine networks can also become a building block of larger networks by implementing another spine layer (sometimes called a "superspine").
Multi-stage Clos networks are quite common, especially in large data centers.

[Section 3.2 of RFC 7938](https://tools.ietf.org/html/rfc7938#section-3.2) provides more information about Clos topologies for the data center.
[Facebook](https://engineering.fb.com/2019/03/14/data-center-engineering/f16-minipack/), [Dropbox](https://dropbox.tech/infrastructure/the-scalable-fabric-behind-our-growing-data-center-network) and [others](../links.md) have written extensively on their network design.

# Fabric Routing

## Summary

Clos-based network fabrics have proven to be a robust yet cost-effective design choice for the physical topology.
This section of the Nokia data center blueprint has attempted to present the main considerations for choosing the routing protocol best suited to your data center fabric.

Understanding the applications that will be deployed on the network will help to determine which protocol is best.
Do any of the applications require IPv4 or IPv6?
Will routing be extended all the way to the end host (for something like Kubernetes)?
The expected size of the network is another important factor.
How many switches will be deployed?
How many links will connect each tier of the network?
Are any routing policies needed (often this is only needed at the boundary between networks)?
Using the answers to these questions, decisions can be made about IP addressing, AS numbering and so forth.
In most cases, it is recommended to build the data center fabric using eBGP and incorporating the best practices outlined in this document.

Our next section will cover the configuration, management and operations of a data center fabric.


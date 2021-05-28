# PMTUD

*PMTUD may not be needed in DC environment if it is recommended to set all MTU for jumbos*

Path MTU Discovery (PMTUD) is a technique used to dynamically discover the MTU size on a network path between two IP hosts.
By determining the Path MTU, protocols such as BGP can maximize packet efficiency for the protocol transmissions and avoid packet fragmentation.
This technique is standardized in RFC 1191 and in for IPv6 in RFC 1981.

To start, the source host assumes the Path MTU of a path is the configured MTU of the directly connected physical interface.
In the first example below in Figure 5, from left to right, the physical interface between the first two nodes is 9212.
The MTU between the 2nd and 3rd, final node is 512.

The nodes send any TCP related datagram to Node 3 on that path with the DF bit set.
Any device along the path whose MTU is smaller than the packet will drop it and send back a response.
In Figure 5 below we see that the 2nd node will send a ICMP “Fragmentation Needed” packet (Type 3, Code 4) or an ICMPv6 ‘packet too big’ message (Type 2, Code 0) respectively.
This exchange of messages allows the source host to reduce its path MTU as needed.
The process repeats until the MTU is small enough to traverse the entire path without fragmentation, end to end.

> insert figure


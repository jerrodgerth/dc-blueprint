# Overall Considerations

When designing a Clos fabric, there are a number of parameters that need to be taken into consideration ranging from the physical layer to the service layer to the application layer.

Here are some questions for consideration:

* Server density per rack and connection speed
* Power budgets per rack, per POD, per DC
* Distance between racks
* Existing cable infrastructure
* What applications will run in the fabric
* Applications that are sensitive to latency and/or packet loss (may determine the physical proximity and/or oversubscription ratio)
* Compute virtualization or containers/micro-services
* How much bandwidth is required per server
* How much traffic is required between servers
* How is storage being connected (Fiber Channel, iSCSI, NFS)

While this isn't an exhaustive list, answers to these questions combined with an understanding of growth plans will allow a proper network to be designed.


## Leaf Considerations

In a Clos network fabric, leaf (ToR) switches are the devices connecting to the end devices.
These end devices are most commonly servers, but could also be security devices, load-balancers or other routers.
Depending on the end device type, the leaf switch may have additional network roles.
In a fabric supporting overlay networks (e.g. EVPN/VXLAN), the leaf switch will commonly be a tunnel end point (TEP).
In the case of VXLAN the leaf switch could be responsible for encap/decap of VXLAN headers.
Whether or not a leaf will be required to be TEP can impact what type of physical switch can used (most but not all network chipsets support this function).

Depending on the level of redundancy required by the attached servers, there often two leaf switches installed in each server rack.
Advantages of having two leaf switches include protection against switch failure, replacement or upgrade.
From the perspective of the server connecting to the leaf switches it is important that logically the switches appear as a single device and the server can attach via a standard 802.3ad trunk.
On the switch side, Nokia has uses the multi-homing capabilities within EVPN to properly forward traffic and prevent network loops.
While not widespread yet, it is also possible to route traffic all the way to the end device.
This configuration would not require a 802.3ad trunk but usually relies on a routing protocol on the server that peers with the leaf switch.
Server connectivity is discussed in more detail in a later section of this blueprint.

A leaf switch is connecting network devices rather than compute or storage is often called a "border leaf."
In the border leaf use case it is also common for the switch to act as a TEP, so selecting the right switch platform is an important consideration.
Other factors such as tunnel scale, route table size and buffers also impact the platform chosen for a border leaf.


## Spine Considerations

Spine switches are responsible for connecting all of the leaf switches.
In the case where a superspine tier is used, spine switches will also connect to one or more of the superspines (depending on the type of design).
Because of this role, it is common to avoid oversubscription.
Oversubscription should be moved to the edges of the network, at the leaf or border leaf with platforms suitable to supporting this requirement.
Based on the size of the network it is also common to consider both fixed-form and chassis switches.
In the last few years, there has been a preference for fixed-form switches to reduce cost as well as complexity.
If buffers are not needed nor TEP functions then spine requirements are quite simple; focused primarily on speed and cost.
As the speed of network ASICs increaases, the network diameter supported by a single ASIC also increases.
For example, a Broadcom Tomahawk3 ASIC supports up to 128 100G ports on a single chip.
This means a simple spine/leaf fabric with Tomahawk3 at the spine could support up to 128 leaf switches.
Factor in multi-stage designs and it is easy to see how the largest networks can use these platforms to scale ecoonomically.

Next, let's consider two example networks.

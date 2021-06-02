# Fabric Management

## Zero Touch Provisioning

* Quick blurb on the need for ZTP
* Highlight the need to remain completely flexible to allow for operator to have complete control over what gets executed and installed during ZTP
* Highlight completely open / blank slate python provisioning script and how enables previous point
	
On traditional network deployments, network administrators required a completely or partially manual multistep process in order to provision global and local parameters and configurations usually involving potential human errors.

Zero Touch Provisioning (ZTP) automatically configures the nodes by obtaining the required information from the network and provisioning them with minimal manual intervention and configuration.
The technician installs the nodes into the rack and when power is applied, and if connectivity is available, the nodes are auto provisioned.


### ZTP Required Components: 

* DHCP server (IPv4 or IPv6) – To support the assignment of IP addresses through DHCP requests and offers.
* File server – for staging and transfer of RPMs, configurations, images, and scripts. HTTP, HTTPS, TFTP, and FTP are supported.
(For HTTPS, the default Mozilla certificate should be used.)
* DHCP relay – required if the server is outside the management interface broadcast domain.


### Support Deployment Models: 

* Nodes, HTTP file servers, and DHCP server in the same subnet (**Figure 4-1**)

<figure>
  <img src="/_images/fig-04-01.png" width="600" />
  <figcaption>Figure 4-1</figcaption>
</figure>

* HTTP file servers and DHCP server in the same subnet, separate from the nodes (**Figure 4-2**)

<figure>
  <img src="/_images/fig-04-02.png" width="600" />
  <figcaption>Figure 4-2</figcaption>
</figure>

* Nodes, HTTP file servers, and DHCP server in different subnets (**Figure 4-3**)

<figure>
  <img src="/_images/fig-04-03.png" width="600" />
  <figcaption>Figure 4-3</figcaption>
</figure>


### Python provisioning script 

The node downloads the Python script and places it on the storage device.
The node then uses the Python provisioning script to download any RPMs, images, scripts, or config and places them in the destination dictated by the script.
This gives the operators total control on what is executed during the ZTP process in terms of software release, complete or partial configurations, automation scripts, etc.


### Configuration Generation

* Blurb about the need for common models driving the entire OS from configuration to operations.  (highlight no difference in data model from CLI to programming models derived from YANG models)
* Everything modeled in YANG allows for data structures to be set in many programming languages and as part of a CI/CD pipeline
* Expand on how this fits into a CI/CD pipeline

A key aspect of programmatically configuring and interacting with a network device is an easily consumable data model; that is, whichever tools or scripts are developed to generate a configuration needs easily consume and populate the NOS data model.
Historically, the NOS CLI has been considered the source of truth for the data model of the NOS.
YANG has emerged as a leading language to model the network operating system data and allow for ease of binding YANG data models into programming language specific objects.
This allows operators to generate a NOS configuration in their preferred programming/scripting language and easily insert this process into a CI/CD pipeline in a truly programmatic way.

> insert sample pipeline diagram here


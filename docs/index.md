# Introduction

Welcome to the Nokia Data Center Blueprint.
The goal of this blueprint is to review many of the important design considerations when building a data center fabric and provide some suggestions.
This will be a living document so new sections will be added over time.
Our initial focus will be on deploying SR Linux for a routed data center fabric.
Topics like building a lab with SR Linux software and using EVPN & VXLAN for network virtualization will be added soon.

This blueprint isn’t meant as a replacement for the full set of User Guides published with each release of SR Linux software.
The User Guide provide great detail for all of the supported features of SR Linux.

### Who should read this?

The target audience is a network architect or SRE (Site Reliability Engineer) looking build a data center fabric using Nokia SR Linux and IXR switches.

## Nokia Data Center Fabric

There are three major components of the Nokia Data Center Fabric solution.
A brief introduction is provided here and more information can be found at <https://www.nokia.com/networks/dc-fabric/>

### SR Linux
Nokia SR Linux is a new NOS developed specifically to meet the needs of hyper scale customers and yet be consumable by any customer regardless of size.

> Nokia SR Linux opens up the future for cloud builder and hyper-scaler data center networking teams.
> Its model-driven foundation is designed from the ground up with a management architecture that meets today’s demands for scalability, visibility, cost-efficiency and ease of operations.
>   
> SR Linux achieves true openness and extensibility by using an unmodified Linux-based kernel as the foundation on which to build and run a suite of network applications.
> This gives network teams reliability, portability and ease of application development — while also speeding the availability of non-Nokia applications.
>
>   -- <https://www.nokia.com/networks/dc-fabric/simplify/>


### IXR Hardware
SR Linux is designed to run on the IXR family of data center switches.

> Our [hardware] portfolio is based on merchant silicon with a common hardware design.
> Products include the Nokia 7220 Interconnect Router (IXR) and Nokia 7250 IXR, offering a broad range of high-performance platforms for data center top of rack (TOR), leaf, spine and super-spine applications.
>   
> Choose between fixed-form-factor and chassis-based platforms, so your data center network teams can select the appropriate hardware while still enjoying all the benefits of running the same SR Linux NOS.
>
>   -- <https://www.nokia.com/networks/dc-fabric/routers/>  

### Fabric Services System

> The Nokia Fabric Services Platform is designed for the intent-based automation of all phases of data center fabric operations, giving your data center networking team all the confidence, efficiency, agility and control they crave.
> Based on an open Kubernetes framework, all fabric services use a distributed microservices approach to deliver a true cloud-native automation and operations platform.
>
> The platform also provides a digital sandbox that is a true emulation of the data center fabric, creating a virtual digital twin of the live production network.
> This emulates a data center fabric and application workloads and can emulate Day 0 design, Day 1 deployment and Day 2+ operations such as operation, fabric change management and troubleshooting.
> 
>   -- <https://www.nokia.com/networks/dc-fabric/automate/>

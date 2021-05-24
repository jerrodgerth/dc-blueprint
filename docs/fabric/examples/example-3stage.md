# 3-Stage Clos Example


### Overview

* A 3-stage Clos and a [5-stage Clos](./example-5stage.md) are provided as examples.  These designs provide the basic concepts and can be extended if needed.
* For simplicity, both examples are built with [7220-IXR-D series](https://onestore.nokia.com/asset/i/207599) switches which are built on the Trident3 chipset.
* These examples are not exhaustive. There are many different ways to build Clos networks depending on the design goals.


### Network Diagram

<figure>
  <img src="/_images/3stage.png" width="600" />
  <figcaption>3-stage Clos</figcaption>
</figure>


### Assumptions


* Uplinks from Leaf to Spine are 100G
* Servers can connect at either 10G or 25G
* Support for single-homed or dual-homed servers (only impact is server density)
* Oversubscription calculations are based on 25G downlink and 100G uplink
    * Ratio could change by adding more Spines (up to 8 max)
* Connections out of the fabric will come via a pair of Leaf nodes (Border Leafs)
* Oversubscription ratio is 1:1 between Spine and Border Leaf


### Details

|                                |                                   |
| :----------------------------- | :-------------------------------- |
| Spine node                     | 7220-IXR-D3 (32xQSFP28 + 2xSFP+)  |
| Leaf node                      | 7220-IXR-D2 (48xSFP28+8xQSFP28)   |
| Border Leaf node               | 7220-IXR-D3 (32xQSFP28 + 2xSFP+)  |
| # of Spines                    | 4                                 |
| # of Leafs                     | 30                                |
| # of Border Leafs              | 2                                 |
| Max single-homed servers       | 1,440 (48*30)                     |
| Max dual-homed servers         | 720 (48*15)                       |
| Leaf-to-spine oversubscription | 3:1                               |

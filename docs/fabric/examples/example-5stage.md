# 5-stage Clos Example


### Network Diagram

<figure>
  <img src="/_images/5stage.png" width="600" />
  <figcaption>5-stage Clos</figcaption>
</figure>


### Assumptions


* Uplinks from Leaf to Spine are 100G
* Servers can connect at either 10G or 25G
* Support for single-homed or dual-homed servers (only impact is server density)
* Leaf to Spine oversubscription calculations are based on 25G downlink and 100G uplink
* Connections out of the fabric will come via a pair of Leaf nodes (Border Leafs)
* Oversubscription ratio is 1:1 between Spine, Superspine and Border Leaf
* Number of Border Leafs can scale out based on amount of North/South traffic


### Details

|                                |                                   |
| :----------------------------- | :-------------------------------- |
| Spine node                     | 7220-IXR-D3 (32xQSFP28 + 2xSFP+)  |
| Leaf node                      | 7220-IXR-D2 (48xSFP28+8xQSFP28)   |
| Border Leaf node               | 7220-IXR-D3 (32xQSFP28 + 2xSFP+)  |
| # of Spines per Pod            | 4                                 |
| # of Leafs per Pod             | 16                                |
| # of Pods                      | 4                                 |
| # of Superspines               | 16                                |
| # of Border Leafs              | 2 (or more)                       |
| Max single-homed servers       | 3,072 (48*64)                     |
| Max dual-homed servers         | 1,536 (48*32)                     |
| Leaf-to-spine oversubscription | 3:1                               |

# Fabric Management

## gNMI

### Overview

gRPC Network Management Interface (gNMI) is a [gRPC](https://www.grpc.io)-based protocol that defines a service or set of services (set of RPC methods) used to configure and retrieve data from network devices.
The gNMI gRPC service is comprised of four RPCs: CAPABILITIES, GET, SET and SUBSCRIBE.
The gNMI spec and associated protobuf defines the type of messages and the data structures used to get, set or stream information to and from network devices.

| Method       | Description |
| ------------ | ----------- |
| CAPABILITIES | Retrieve the set of capabilities that is supported by the server. This allows the client to validate the service version that is implemented and retrieve the set of models that the server supports. The models can then be specified in subsequent RPCs to restrict the set of data that is utilized. |
| GET          | Retrieve a snapshot of data from the server. A Get RPC requests that the server snapshots a subset of the data tree as specified by the paths included in the message and serializes this to be returned to the client using the specified encoding. |
| SET          | Modify the state of data on the server. The paths to modified along with the new values that the client wishes to set the value to. |
| SUBSCRIBE    | Request the server to send it values of particular paths within the data tree. These values may be streamed at a particular cadence (STREAM), sent one off on a long-lived channel (POLL), or sent as a one-off retrieval (ONCE). |

### gNMI Subscription

A gNMI Subscription to a server consists of the following key attributes:

* One or multiple paths to subscribe to (what data should be sent to client by server)
* A subscription mode  

gNMI SUBSCRIBE RPC defines three subscription modes:

* **ONCE:** A subscription to a server is created and a single request is sent.  The server creates an update message for the requested path and sends it to the client.  The subscription is then terminated.
* **POLL:** A subscription to a server is created for a given path, the client will then send a specific message to the server on a timed interval, upon receipt of this message the server will send the data for the path on which the subscription is created.
* **STREAM:** A subscription to a server is created for a given path, the server is then expected to send data back to the client for that given path in one of three ways, `on_change`, `on_sample` or `target_defined`


### gNMI Server Configuration

```shell
--{ + running }--[ system gnmi-server ]--
A:srl2# info detail
    admin-state enable
    timeout 7200
    rate-limit 60
    session-limit 20
    commit-confirmed-timeout 0
    include-defaults-in-config-only-responses false
    network-instance mgmt {
        admin-state enable
        use-authentication true
        port 57400
        tls-profile tls-profile-1
    }
    unix-socket {
        admin-state disable
        use-authentication true
    }
```

Configuration explanation:

**`admin-state`:** enabled under the global gNMI context it will allow for gNMI to be enabled under network-instances or unix-socket.
If disabled, all gNMI servers will be disabled.

**`timeout`:** specify the number of seconds the a gNMI connection will remain idle before closing the connection 

**`rate-limit`:** set the max number of connection attempts allowed per minute

**`session-limit`:** sets the limit on the number of simultaneous active gNMI sessions

**`network-instance`:** specify the network-instance under which a gNMI server should be reachable

**`admin-state`:** enabled under the network-instance or unix-socket in order to enable or disable an instance of the gNMI server under a given network-instance

**`use-authentication`:** specifies whether the RPC should be authenticated against a user using aaa_mgr

**`port`:** specifies the port on which the gNMI server should listen, by default it will listen on 57400

**`tls-profile`:** a TLS profile must be configured, see TLS workshop for details around configuring the profile itself.

**`source-address`:** specifies an IP address which the gNMI server will listen on, the IP address must be present in the given network-instance.
The IP address can be either IPv4 or IPv6.
If set to 0.0.0.0 it will listen on any address within the network-instance for IPv4.
If set to :: then it will listen on all IPv4 and IPv6 addresses.

**`unix-socket`:** specifies whether the RPC should be authenticated against a user using aaa_mgr


### gNMI Examples

!!!note
    The [gNMIc](https://gnmic.kmrd.dev) client is used for each of these examples.  gNMIc is one of several open source gNMI clients available.


#### GET Example

```shell
demo@demo-sf03:~$ gnmic -a 172.20.20.2:57400 -e json_ietf -u admin -p admin --skip-verify get --path "/system/name/host-name"

Get Response:
[
  {
    "timestamp": 1604412647649136431,
    "time": "2020-11-03T15:10:47.649136431+01:00",
    "updates": [
      {
        "Path": "srl_nokia-system:system/srl_nokia-system-name:name/host-name",
        "values": {
          "srl_nokia-system:system/srl_nokia-system-name:name/host-name": "srl1"
        }
      }
    ]
  }
]
```


#### SET Example

```shell
demo@demo-sf03:~$ gnmic -a 172.20.20.2:57400 -e json_ietf -u admin -p admin --skip-verify set --replace /system/name/host-name:::string:::test1

Set Response:
{
  "timestamp": 1604412698946981257,
  "time": "2020-11-03T15:11:38.946981257+01:00",
  "results": [
    {
      "operation": "REPLACE",
      "path": "system/name/host-name"
    }
  ]
}
```


#### SUBSCRIBE `once` Example

```shell
demo@demo-sf03:~$ gnmic -a 172.20.20.2:57400 -e json_ietf -u admin -p admin --skip-verify subscribe --mode once --path "/system/name/host-name"

{
  "source": "srl1:57400",
  "subscription-name": "default",
  "timestamp": 1604412764363675969,
  "time": "2020-11-03T15:12:44.363675969+01:00",
  "updates": [
    {
      "Path": "srl_nokia-system:system/srl_nokia-system-name:name/host-name",
      "values": {
        "srl_nokia-system:system/srl_nokia-system-name:name/host-name": "srl1"
      }
    }
  ]
}
```


#### SUBSCRIBE `on_change` Example

```shell
demo@demo-sf03:~$ gnmic -a 172.20.20.2:57400 -e json_ietf -u admin -p admin --skip-verify subscribe --mode stream --stream-mode on_change --path "/interface[name=ethernet-1/1]/statistics/in-octets"

{
  "source": "srl1:57400",
  "subscription-name": "default",
  "timestamp": 1604412831614958694,
  "time": "2020-11-03T15:13:51.614958694+01:00",
  "updates": [
    {
      "Path": "srl_nokia-interfaces:interface[name=ethernet-1/1]/statistics/in-octets",
      "values": {
        "srl_nokia-interfaces:interface/statistics/in-octets": "16170673"
      }
    }
  ]
}
{
  "source": "srl1:57400",
  "subscription-name": "default",
  "timestamp": 1604412833881891643,
  "time": "2020-11-03T15:13:53.881891643+01:00",
  "updates": [
    {
      "Path": "srl_nokia-interfaces:interface[name=ethernet-1/1]/statistics/in-octets",
      "values": {
        "srl_nokia-interfaces:interface/statistics/in-octets": "16170778"
      }
    }
  ]
}
```

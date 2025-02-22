# Network configuration

LXD supports the following network types:

 - [bridge](#network-bridge): Creates an L2 bridge for connecting instances to (can provide local DHCP and DNS). This is the default.
 - [macvlan](#network-macvlan): Provides preset configuration to use when connecting instances to a parent macvlan interface.
 - [sriov](#network-sriov): Provides preset configuration to use when connecting instances to a parent SR-IOV interface.
 - [ovn](#network-ovn): Creates a logical network using the OVN software defined networking system.
 - [physical](#network-physical): Provides preset configuration to use when connecting OVN networks to a parent interface.

The desired type can be specified using the `--type` argument, e.g.

```bash
lxc network create <name> --type=bridge [options...]
```

If no `--type` argument is specified, the default type of `bridge` is used.

The configuration keys are namespaced with the following namespaces currently supported for all network types:

 - `maas` (MAAS network identification)
 - `user` (free form key/value for user metadata)

## network: bridge

As one of the possible network configuration types under LXD, LXD supports creating and managing network bridges.
LXD bridges can leverage underlying native Linux bridges and Open vSwitch.

Creation and management of LXD bridges is performed via the `lxc network` command.
A bridge created by LXD is by default "managed" which means that LXD also will additionally set up a local `dnsmasq`
DHCP server and if desired also perform NAT for the bridge (this is the default.)

When a bridge is managed by LXD, configuration values under the `bridge` namespace can be used to configure it.

Additionally, LXD can utilize a pre-existing Linux bridge. In this case, the bridge does not need to be created via
`lxc network` and can simply be referenced in an instance or profile device configuration as follows:

```
devices:
  eth0:
     name: eth0
     nictype: bridged
     parent: br0
     type: nic
```

Network forwards:

Bridge networks support [network forwards](network-forwards.md#network-bridge).

Network configuration properties:

A complete list of configuration settings for LXD networks can be found below.

The following configuration key namespaces are currently supported for bridge networks:

 - `bridge` (L2 interface configuration)
 - `fan` (configuration specific to the Ubuntu FAN overlay)
 - `tunnel` (cross-host tunneling configuration)
 - `ipv4` (L3 IPv4 configuration)
 - `ipv6` (L3 IPv6 configuration)
 - `dns` (DNS server and resolution configuration)
 - `raw` (raw configuration file content)

It is expected that IP addresses and subnets are given using CIDR notation (`1.1.1.1/24` or `fd80:1234::1/64`).

The exception being tunnel local and remote addresses which are just plain addresses (`1.1.1.1` or `fd80:1234::1`).

Key                                  | Type      | Condition             | Default                   | Description
:--                                  | :--       | :--                   | :--                       | :--
bgp.peers.NAME.address               | string    | bgp server            | -                         | Peer address (IPv4 or IPv6)
bgp.peers.NAME.asn                   | integer   | bgp server            | -                         | Peer AS number
bgp.peers.NAME.password              | string    | bgp server            | - (no password)           | Peer session password (optional)
bgp.ipv4.nexthop                     | string    | bgp server            | local address             | Override the next-hop for advertised prefixes
bgp.ipv6.nexthop                     | string    | bgp server            | local address             | Override the next-hop for advertised prefixes
bridge.driver                        | string    | -                     | native                    | Bridge driver ("native" or "openvswitch")
bridge.external\_interfaces          | string    | -                     | -                         | Comma separate list of unconfigured network interfaces to include in the bridge
bridge.hwaddr                        | string    | -                     | -                         | MAC address for the bridge
bridge.mode                          | string    | -                     | standard                  | Bridge operation mode ("standard" or "fan")
bridge.mtu                           | integer   | -                     | 1500                      | Bridge MTU (default varies if tunnel or fan setup)
dns.domain                           | string    | -                     | lxd                       | Domain to advertise to DHCP clients and use for DNS resolution
dns.mode                             | string    | -                     | managed                   | DNS registration mode ("none" for no DNS record, "managed" for LXD generated static records or "dynamic" for client generated records)
dns.search                           | string    | -                     | -                         | Full comma separated domain search list, defaulting to `dns.domain` value
dns.zone.forward                     | string    | -                     | managed                   | DNS zone name for forward DNS records
dns.zone.reverse.ipv4                | string    | -                     | managed                   | DNS zone name for IPv4 reverse DNS records
dns.zone.reverse.ipv6                | string    | -                     | managed                   | DNS zone name for IPv6 reverse DNS records
fan.overlay\_subnet                  | string    | fan mode              | 240.0.0.0/8               | Subnet to use as the overlay for the FAN (CIDR notation)
fan.type                             | string    | fan mode              | vxlan                     | The tunneling type for the FAN ("vxlan" or "ipip")
fan.underlay\_subnet                 | string    | fan mode              | auto (on create only)     | Subnet to use as the underlay for the FAN (CIDR notation). Use "auto" to use default gateway subnet
ipv4.address                         | string    | standard mode         | auto (on create only)     | IPv4 address for the bridge (CIDR notation). Use "none" to turn off IPv4 or "auto" to generate a new random unused subnet
ipv4.dhcp                            | boolean   | ipv4 address          | true                      | Whether to allocate addresses using DHCP
ipv4.dhcp.expiry                     | string    | ipv4 dhcp             | 1h                        | When to expire DHCP leases
ipv4.dhcp.gateway                    | string    | ipv4 dhcp             | ipv4.address              | Address of the gateway for the subnet
ipv4.dhcp.ranges                     | string    | ipv4 dhcp             | all addresses             | Comma separated list of IP ranges to use for DHCP (FIRST-LAST format)
ipv4.firewall                        | boolean   | ipv4 address          | true                      | Whether to generate filtering firewall rules for this network
ipv4.nat.address                     | string    | ipv4 address          | -                         | The source address used for outbound traffic from the bridge
ipv4.nat                             | boolean   | ipv4 address          | false                     | Whether to NAT (defaults to true for regular bridges where ipv4.address is generated and always defaults to true for fan bridges)
ipv4.nat.order                       | string    | ipv4 address          | before                    | Whether to add the required NAT rules before or after any pre-existing rules
ipv4.ovn.ranges                      | string    | -                     | -                         | Comma separate list of IPv4 ranges to use for child OVN network routers (FIRST-LAST format)
ipv4.routes                          | string    | ipv4 address          | -                         | Comma separated list of additional IPv4 CIDR subnets to route to the bridge
ipv4.routing                         | boolean   | ipv4 address          | true                      | Whether to route traffic in and out of the bridge
ipv6.address                         | string    | standard mode         | auto (on create only)     | IPv6 address for the bridge (CIDR notation). Use "none" to turn off IPv6 or "auto" to generate a new random unused subnet
ipv6.dhcp                            | boolean   | ipv6 address          | true                      | Whether to provide additional network configuration over DHCP
ipv6.dhcp.expiry                     | string    | ipv6 dhcp             | 1h                        | When to expire DHCP leases
ipv6.dhcp.ranges                     | string    | ipv6 stateful dhcp    | all addresses             | Comma separated list of IPv6 ranges to use for DHCP (FIRST-LAST format)
ipv6.dhcp.stateful                   | boolean   | ipv6 dhcp             | false                     | Whether to allocate addresses using DHCP
ipv6.firewall                        | boolean   | ipv6 address          | true                      | Whether to generate filtering firewall rules for this network
ipv6.nat.address                     | string    | ipv6 address          | -                         | The source address used for outbound traffic from the bridge
ipv6.nat                             | boolean   | ipv6 address          | false                     | Whether to NAT (will default to true if unset and a random ipv6.address is generated)
ipv6.nat.order                       | string    | ipv6 address          | before                    | Whether to add the required NAT rules before or after any pre-existing rules
ipv6.ovn.ranges                      | string    | -                     | -                         | Comma separate list of IPv6 ranges to use for child OVN network routers (FIRST-LAST format)
ipv6.routes                          | string    | ipv6 address          | -                         | Comma separated list of additional IPv6 CIDR subnets to route to the bridge
ipv6.routing                         | boolean   | ipv6 address          | true                      | Whether to route traffic in and out of the bridge
maas.subnet.ipv4                     | string    | ipv4 address          | -                         | MAAS IPv4 subnet to register instances in (when using `network` property on nic)
maas.subnet.ipv6                     | string    | ipv6 address          | -                         | MAAS IPv6 subnet to register instances in (when using `network` property on nic)
raw.dnsmasq                          | string    | -                     | -                         | Additional dnsmasq configuration to append to the configuration file
tunnel.NAME.group                    | string    | vxlan                 | 239.0.0.1                 | Multicast address for vxlan (used if local and remote aren't set)
tunnel.NAME.id                       | integer   | vxlan                 | 0                         | Specific tunnel ID to use for the vxlan tunnel
tunnel.NAME.interface                | string    | vxlan                 | -                         | Specific host interface to use for the tunnel
tunnel.NAME.local                    | string    | gre or vxlan          | -                         | Local address for the tunnel (not necessary for multicast vxlan)
tunnel.NAME.port                     | integer   | vxlan                 | 0                         | Specific port to use for the vxlan tunnel
tunnel.NAME.protocol                 | string    | standard mode         | -                         | Tunneling protocol ("vxlan" or "gre")
tunnel.NAME.remote                   | string    | gre or vxlan          | -                         | Remote address for the tunnel (not necessary for multicast vxlan)
tunnel.NAME.ttl                      | integer   | vxlan                 | 1                         | Specific TTL to use for multicast routing topologies
security.acls                        | string    | -                     | -                         | Comma separated list of Network ACLs to apply to NICs connected to this network (see [Limitations](network-acls.md#bridge-limitations))
security.acls.default.ingress.action | string    | security.acls         | reject                    | Action to use for ingress traffic that doesn't match any ACL rule
security.acls.default.egress.action  | string    | security.acls         | reject                    | Action to use for egress traffic that doesn't match any ACL rule
security.acls.default.ingress.logged | boolean   | security.acls         | false                     | Whether to log ingress traffic that doesn't match any ACL rule
security.acls.default.egress.logged  | boolean   | security.acls         | false                     | Whether to log egress traffic that doesn't match any ACL rule
Those keys can be set using the lxc tool with:

```bash
lxc network set <network> <key> <value>
```

### Integration with systemd-resolved
If the system running LXD uses systemd-resolved to perform DNS
lookups, it's possible to notify resolved of the domain(s) that
LXD is able to resolve.  This requires telling resolved the
specific bridge(s), nameserver address(es), and dns domain(s).

For example, if LXD is using the `lxdbr0` interface, get the
ipv4 address with `lxc network get lxdbr0 ipv4.address` command
(the ipv6 can be used instead or in addition), and the domain
with `lxc network get lxdbr0 dns.domain` (if unset, the domain
is `lxd` as shown in the table above).  Then notify resolved:

```
systemd-resolve --interface lxdbr0 --set-domain '~lxd' --set-dns n.n.n.n
```

Replace `lxdbr0` with the actual bridge name, and `n.n.n.n` with
the actual address of the nameserver (without the subnet netmask).

Also replace `lxd` with the domain name.  Note the `~` before the
domain name is important; it tells resolved to use this
nameserver to look up only this domain; no matter what your
actual domain name is, you should prefix it with `~`.  Also,
since the shell may expand the `~` character, you may need to
include it in quotes.

In newer releases of systemd, the `systemd-resolve` command has been
deprecated, however it is still provided for backwards compatibility
(as of this writing).  The newer method to notify resolved is using
the `resolvectl` command, which would be done in two steps:

```
resolvectl dns lxdbr0 n.n.n.n
resolvectl domain lxdbr0 '~lxd'
```

This resolved configuration will persist as long as the bridge
exists, so you must repeat this command each reboot and after
LXD is restarted (see below on how to automate this).

Also note this only works if the bridge `dns.mode` is not `none`.

Note that depending on the `dns.domain` used, you may need to disable
DNSSEC in resolved to allow for DNS resolution. This can be done through
the `DNSSEC` option in `resolved.conf`.

To automate the `systemd-resolved` DNS configuration when LXD creates the `lxdbr0` interface so that it is applied
on system start you need to create a systemd unit file `/etc/systemd/system/lxd-dns-lxdbr0.service` containing:

```
[Unit]
Description=LXD per-link DNS configuration for lxdbr0
BindsTo=sys-subsystem-net-devices-lxdbr0.device
After=sys-subsystem-net-devices-lxdbr0.device

[Service]
Type=oneshot
ExecStart=/usr/bin/resolvectl dns lxdbr0 n.n.n.n
ExecStart=/usr/bin/resolvectl domain lxdbr0 '~lxd'

[Install]
WantedBy=sys-subsystem-net-devices-lxdbr0.device
```

Be sure to replace `n.n.n.n` in that file with the IP of the `lxdbr0` bridge.

Then enable and start it using:

```
sudo systemctl daemon-reload
sudo systemctl enable --now lxd-dns-lxdbr0
```

If the `lxdbr0` interface already exists (i.e LXD is running), then you can check that the new service has started:

```
sudo systemctl status lxd-dns-lxdbr0.service
● lxd-dns-lxdbr0.service - LXD per-link DNS configuration for lxdbr0
     Loaded: loaded (/etc/systemd/system/lxd-dns-lxdbr0.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Mon 2021-06-14 17:03:12 BST; 1min 2s ago
    Process: 9433 ExecStart=/usr/bin/resolvectl dns lxdbr0 n.n.n.n (code=exited, status=0/SUCCESS)
    Process: 9434 ExecStart=/usr/bin/resolvectl domain lxdbr0 ~lxd (code=exited, status=0/SUCCESS)
   Main PID: 9434 (code=exited, status=0/SUCCESS)
```

You can then check it has applied the settings using:

```
sudo resolvectl status lxdbr0
Link 6 (lxdbr0)
      Current Scopes: DNS
DefaultRoute setting: no
       LLMNR setting: yes
MulticastDNS setting: no
  DNSOverTLS setting: no
      DNSSEC setting: no
    DNSSEC supported: no
  Current DNS Server: n.n.n.n
         DNS Servers: n.n.n.n
          DNS Domain: ~lxd
```

### IPv6 prefix size
For optimal operation, a prefix size of 64 is preferred.
Larger subnets (prefix smaller than 64) should work properly too but
aren't typically that useful for SLAAC.

Smaller subnets while in theory possible when using stateful DHCPv6 for
IPv6 allocation aren't properly supported by dnsmasq and may be the
source of issue. If you must use one of those, static allocation or
another standalone RA daemon be used.

### Allow DHCP, DNS with Firewalld

In order to allow instances to access the DHCP and DNS server that LXD runs on the host when using firewalld
you need to add the host's bridge interface to the `trusted` zone in firewalld.

To do this permanently (so that it persists after a reboot) run the following command:

```
firewall-cmd --zone=trusted --change-interface=<LXD network name> --permanent
```

E.g. for a bridged network called `lxdbr0` run the command:

```
firewall-cmd --zone=trusted --change-interface=lxdbr0 --permanent
```

This will then allow LXD's own firewall rules to take effect.


### How to let Firewalld control the LXD's iptables rules

When using firewalld and LXD together, iptables rules can overlaps. For example, firewalld could erase LXD iptables rules if it is started after LXD daemon, then LXD container will not be able to do any oubound internet access.
One way to fix it is to delegate to firewalld the LXD's iptables rules and to disable the LXD ones.

First step is to [allow DNS and DHCP](#allow-dhcp-dns-with-firewalld).

Then to tell to LXD totally stop to set iptables rules (because firewalld will do it):
```
lxc network set lxdbr0 ipv4.nat false
lxc network set lxdbr0 ipv6.nat false
lxc network set lxdbr0 ipv6.firewall false
lxc network set lxdbr0 ipv4.firewall false
```

Finally, to enable iptables firewalld's rules for LXD usecase (in this example, we suppose the bridge interface is `lxdbr0` and the associated IP range is `10.0.0.0/24`:
```
firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT 0 -i lxdbr0 -s 10.0.0.0/24 -m comment --comment "generated by firewalld for LXD" -j ACCEPT
firewall-cmd --permanent --direct --add-rule ipv4 filter OUTPUT 0 -o lxdbr0 -d 10.0.0.0/24 -m comment --comment "generated by firewalld for LXD" -j ACCEPT
firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -i lxdbr0 -s 10.0.0.0/24 -m comment --comment "generated by firewalld for LXD" -j ACCEPT
firewall-cmd --permanent --direct --add-rule ipv4 nat POSTROUTING 0 -s 10.0.0.0/24 ! -d 10.0.0.0/24 -m comment --comment "generated by firewalld for LXD" -j MASQUERADE
firewall-cmd --reload
```
To check the rules are taken into account by firewalld:
```
firewall-cmd --direct --get-all-rules
```

Warning: what is exposed above is not a fool-proof approach and may end up inadvertently introducing a security risk.

## network: macvlan

The macvlan network type allows one to specify presets to use when connecting instances to a parent interface
using macvlan NICs. This allows the instance NIC itself to simply specify the `network` it is connecting to without
knowing any of the underlying configuration details.

Network configuration properties:

Key                             | Type      | Condition             | Default                   | Description
:--                             | :--       | :--                   | :--                       | :--
maas.subnet.ipv4                | string    | ipv4 address          | -                         | MAAS IPv4 subnet to register instances in (when using `network` property on nic)
maas.subnet.ipv6                | string    | ipv6 address          | -                         | MAAS IPv6 subnet to register instances in (when using `network` property on nic)
mtu                             | integer   | -                     | -                         | The MTU of the new interface
parent                          | string    | -                     | -                         | Parent interface to create macvlan NICs on
vlan                            | integer   | -                     | -                         | The VLAN ID to attach to
gvrp                            | boolean   | -                     | false                     | Register VLAN using GARP VLAN Registration Protocol

## network: sriov

The sriov network type allows one to specify presets to use when connecting instances to a parent interface
using sriov NICs. This allows the instance NIC itself to simply specify the `network` it is connecting to without
knowing any of the underlying configuration details.

Network configuration properties:

Key                             | Type      | Condition             | Default                   | Description
:--                             | :--       | :--                   | :--                       | :--
maas.subnet.ipv4                | string    | ipv4 address          | -                         | MAAS IPv4 subnet to register instances in (when using `network` property on nic)
maas.subnet.ipv6                | string    | ipv6 address          | -                         | MAAS IPv6 subnet to register instances in (when using `network` property on nic)
mtu                             | integer   | -                     | -                         | The MTU of the new interface
parent                          | string    | -                     | -                         | Parent interface to create sriov NICs on
vlan                            | integer   | -                     | -                         | The VLAN ID to attach to

## network: ovn

The ovn network type allows the creation of logical networks using the OVN SDN. This can be useful for labs and
multi-tenant environments where the same logical subnets are used in multiple discrete networks.

A LXD OVN network can be connected to an existing managed LXD bridge network in order for it to gain outbound
access to the wider network. All connections from the OVN logical networks are NATed to a dynamic IP allocated by
the parent network.

### Standalone LXD OVN setup

This will create a standalone OVN network that is connected to the parent network lxdbr0 for outbound connectivity.

Install the OVN tools and configure the OVN integration bridge on the local node:

```
sudo apt install ovn-host ovn-central
sudo ovs-vsctl set open_vswitch . \
  external_ids:ovn-remote=unix:/var/run/ovn/ovnsb_db.sock \
  external_ids:ovn-encap-type=geneve \
  external_ids:ovn-encap-ip=127.0.0.1
```

Create an OVN network and an instance using it:

```
lxc network set lxdbr0 ipv4.dhcp.ranges=... ipv4.ovn.ranges=... # Allocate IP range for OVN gateways.
lxc network create ovntest --type=ovn network=lxdbr0
lxc init images:ubuntu/20.04 c1
lxc config device override c1 eth0 network=ovntest
lxc start c1
lxc ls
+------+---------+---------------------+----------------------------------------------+-----------+-----------+
| NAME |  STATE  |        IPV4         |                     IPV6                     |   TYPE    | SNAPSHOTS |
+------+---------+---------------------+----------------------------------------------+-----------+-----------+
| c1   | RUNNING | 10.254.118.2 (eth0) | fd42:887:cff3:5089:216:3eff:fef0:549f (eth0) | CONTAINER | 0         |
+------+---------+---------------------+----------------------------------------------+-----------+-----------+
```

Network forwards:

OVN networks support [network forwards](network-forwards.md#network-ovn).

Network peers:

OVN networks support [network peers](network-peers.md).

Network configuration properties:

Key                                  | Type      | Condition             | Default                   | Description
:--                                  | :--       | :--                   | :--                       | :--
bridge.hwaddr                        | string    | -                     | -                         | MAC address for the bridge
bridge.mtu                           | integer   | -                     | 1442                      | Bridge MTU (default allows host to host geneve tunnels)
dns.domain                           | string    | -                     | lxd                       | Domain to advertise to DHCP clients and use for DNS resolution
dns.search                           | string    | -                     | -                         | Full comma separated domain search list, defaulting to `dns.domain` value
dns.zone.forward                     | string    | -                     | -                         | DNS zone name for forward DNS records
dns.zone.reverse.ipv4                | string    | -                     | -                         | DNS zone name for IPv4 reverse DNS records
dns.zone.reverse.ipv6                | string    | -                     | -                         | DNS zone name for IPv6 reverse DNS records
ipv4.address                         | string    | standard mode         | auto (on create only)     | IPv4 address for the bridge (CIDR notation). Use "none" to turn off IPv4 or "auto" to generate a new random unused subnet
ipv4.dhcp                            | boolean   | ipv4 address          | true                      | Whether to allocate addresses using DHCP
ipv4.nat                             | boolean   | ipv4 address          | false                     | Whether to NAT (will default to true if unset and a random ipv4.address is generated)
ipv4.nat.address                     | string    | ipv4 address          | -                         | The source address used for outbound traffic from the network (requires uplink `ovn.ingress_mode=routed`)
ipv6.address                         | string    | standard mode         | auto (on create only)     | IPv6 address for the bridge (CIDR notation). Use "none" to turn off IPv6 or "auto" to generate a new random unused subnet
ipv6.nat.address                     | string    | ipv6 address          | -                         | The source address used for outbound traffic from the network (requires uplink `ovn.ingress_mode=routed`)
ipv6.dhcp                            | boolean   | ipv6 address          | true                      | Whether to provide additional network configuration over DHCP
ipv6.dhcp.stateful                   | boolean   | ipv6 dhcp             | false                     | Whether to allocate addresses using DHCP
ipv6.nat                             | boolean   | ipv6 address          | false                     | Whether to NAT (will default to true if unset and a random ipv6.address is generated)
network                              | string    | -                     | -                         | Uplink network to use for external network access
security.acls                        | string    | -                     | -                         | Comma separated list of Network ACLs to apply to NICs connected to this network
security.acls.default.ingress.action | string    | security.acls         | reject                    | Action to use for ingress traffic that doesn't match any ACL rule
security.acls.default.egress.action  | string    | security.acls         | reject                    | Action to use for egress traffic that doesn't match any ACL rule
security.acls.default.ingress.logged | boolean   | security.acls         | false                     | Whether to log ingress traffic that doesn't match any ACL rule
security.acls.default.egress.logged  | boolean   | security.acls         | false                     | Whether to log egress traffic that doesn't match any ACL rule

## network: physical

The physical network type allows one to specify presets to use when connecting OVN networks to a parent interface.

Network configuration properties:

Key                             | Type      | Condition             | Default                   | Description
:--                             | :--       | :--                   | :--                       | :--
bgp.peers.NAME.address          | string    | bgp server            | -                         | Peer address (IPv4 or IPv6) for use by `ovn` downstream networks
bgp.peers.NAME.asn              | integer   | bgp server            | -                         | Peer AS number for use by `ovn` downstream networks
bgp.peers.NAME.password         | string    | bgp server            | - (no password)           | Peer session password (optional) for use by `ovn` downstream networks
maas.subnet.ipv4                | string    | ipv4 address          | -                         | MAAS IPv4 subnet to register instances in (when using `network` property on nic)
maas.subnet.ipv6                | string    | ipv6 address          | -                         | MAAS IPv6 subnet to register instances in (when using `network` property on nic)
mtu                             | integer   | -                     | -                         | The MTU of the new interface
parent                          | string    | -                     | -                         | Parent interface to create sriov NICs on
vlan                            | integer   | -                     | -                         | The VLAN ID to attach to
gvrp                            | boolean   | -                     | false                     | Register VLAN using GARP VLAN Registration Protocol
ipv4.gateway                    | string    | standard mode         | -                         | IPv4 address for the gateway and network (CIDR notation)
ipv4.ovn.ranges                 | string    | -                     | -                         | Comma separate list of IPv4 ranges to use for child OVN network routers (FIRST-LAST format)
ipv4.routes                     | string    | ipv4 address          | -                         | Comma separated list of additional IPv4 CIDR subnets that can be used with child OVN networks ipv4.routes.external setting
ipv4.routes.anycast             | boolean   | ipv4 address          | false                     | Allow the overlapping routes to be used on multiple networks/NIC at the same time.
ipv6.gateway                    | string    | standard mode         | -                         | IPv6 address for the gateway and network  (CIDR notation)
ipv6.ovn.ranges                 | string    | -                     | -                         | Comma separate list of IPv6 ranges to use for child OVN network routers (FIRST-LAST format)
ipv6.routes                     | string    | ipv6 address          | -                         | Comma separated list of additional IPv6 CIDR subnets that can be used with child OVN networks ipv6.routes.external setting
ipv6.routes.anycast             | boolean   | ipv6 address          | false                     | Allow the overlapping routes to be used on multiple networks/NIC at the same time.
dns.nameservers                 | string    | standard mode         | -                         | List of DNS server IPs on physical network
ovn.ingress\_mode               | string    | standard mode         | l2proxy                   | Sets the method that OVN NIC external IPs will be advertised on uplink network. Either `l2proxy` (proxy ARP/NDP) or `routed`.

## BGP integration
LXD can act as a BGP server, effectively allowing to establish sessions with upstream BGP routers and announce the addresses and subnets that it's using.

This can be used to allow a LXD server or cluster to directly use internal/external address space, getting the specific subnets or addresses routed to the correct host for it to forward onto the target instance.

For this to work, `core.bgp_address`, `core.bgp_asn` and `core.bgp_routerid` must be set.
Once those are set, LXD will start listening for BGP sessions.

Peers can be defined on both `bridged` and `physical` managed networks. Additionally in the `bridged` case, a set of per-server configuration keys are also available to override the next-hop. When those aren't specified, the next-hop defaults to the address used for the BGP session.

The `physical` network case is used for `ovn` networks where the uplink network is the one holding the list of allowed subnets and the BGP configuration. Once that parent network is configured, children OVN networks will get their external subnets and addresses announced over BGP with the next-hop set to the OVN router address for the network in question.

The addresses and networks currently being advertised are:
 - Network `ipv4.address` or `ipv6.address` subnets when the matching `nat` property isn't set to `true`
 - Network `ipv4.nat.address` and `ipv6.nat.address` when those are set
 - Instance NIC routes defined through `ipv4.routes.external` or `ipv6.routes.external`

At this time, there isn't a way to only announce some specific routes/addresses to particular peers. Instead it's currently recommended to filter prefixes on the upstream routers.

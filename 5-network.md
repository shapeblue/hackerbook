# Hacking Networking

Outline: (self-study, refer docs http://docs.cloudstack.apache.org/)

  - Overview
  - Basic Vs Advanced Zone
  - Network Models: L2, Shared, Isolated, VPC
  - Isolation: SG, VLAN, VXLAN etc.
  - Virtual Router
  - Network tools, usage and debugging

Reading list:
- Switching and Routing: https://www.practicalnetworking.net/series/packet-traveling/packet-traveling/
- ARP: https://www.practicalnetworking.net/series/arp/address-resolution-protocol/
- NAT, DNAT, SNAT: https://www.practicalnetworking.net/series/nat/nat/
- VLAN: https://www.practicalnetworking.net/stand-alone/vlans/
- Linux Virtual Networking 101: https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking/
- https://www.shapeblue.com/a-beginners-guide-to-cloudstack-networking/

Recommended learning:
- CloudStack Networking series by Chiradeep: https://www.youtube.com/playlist?list=PLDF5F6BBB1C3A7CDD
- Inside CloudStack VR by Rohit: https://docs.google.com/presentation/d/1fTfOaur4BNTStd_NCuwNEGGJ3E4xbf-aTFIP0nqGCiY/edit#slide=id.g38be540065_6_22
- [Networking Refresh by Dag](primer/networking-refresh.pdf)

### Sessions

The following session structure (self-learning or with a colleague/mentor) can be used:

Session 1:
- Basic terms, terminologies
- OSI layer, L1-3, L4-7 layers
- Basic devices (switch, router, bridge, tap, tun, etc.)
- Basic protocols and addressing (arp, dhcp, dns, tcp, udp, icmp, igmp, ipv4, ipv6, optional: ospf, bgp)
- Isolation: vlan, vxlan, sg
- Bridge networking
- CloudStack network models basics
- Practical demo using monkeybox

Session 2:
- Revisit layers
- CloudStack network models adv. with diagrams for each type
- Linux nf framework (netfilters)
- Network tools (iproute2, iptables/ebtables/nftables, tcpdump, ping/arping, netstat, nslookup, arp, traceroute etc.)
- Debugging network stack across machines

Session 3:
- CloudStack SystemVM building, patching, init
- CloudStack agent framework, CPVM/SSVM use-cases

Session 4:
- Practical network
- CloudStack VR programming part
- Implement X: Wrap up, end to end demo and walkthrough
- Debug, find/extend feature etc.

Misc wiki reading:

https://cwiki.apache.org/confluence/display/CLOUDSTACK/CloudStack+Advanced+Network+Tutorial+-+Step+by+Step

https://cwiki.apache.org/confluence/display/CLOUDSTACK/Network+Manager+refactoring

https://cwiki.apache.org/confluence/display/CLOUDSTACK/Refactoring+Redundant+Virtual+Router+Implementation

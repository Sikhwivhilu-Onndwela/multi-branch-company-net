#  Multi-Branch Company Network (Cisco Packet Tracer)

A small company network with a head office (HQ) and two branches, built in Cisco Packet Tracer. It is modelled on a retail or betting-style business: staff PCs, POS terminals, guest access and a , management network, with a DNS server at head office.

**Skills shown:** VLANs and trunking, inter-VLAN routing (router-on-a-stick), DHCP, DNS, static routing between sites, access control lists (ACLs), SSH, port-security, network documentation, ping/traceroute verification.

## Topology
![Topology](screenshots/topology.png)

## Scope and design  decisions
- Head office has four VLANs on one switch. Each branch has a single LAN.
- Sites are joined by two point-to-point links ad static routes. 
- NAT and internet access are not included.
- The Guest VLAN s blocked from internal networks by an ACL, but can still use DNS.

## Addressing

### VLANs and LAN
| Site | VLAN | Name | Subnet | Gateway |
|---|---|---|---|---|
| HQ | 10 | Staff | 192.168.10.0/24 | 192.168.10.1 |
|HQ | 20 | POS | 192.168.20.0/24 | 192.168.20.1 |
| HQ | 30 | Guest | 192.168.30.0/24 | 192.168.30.1 |
| HQ | 99 | Management | 192.168.99.0/24 | 192.168.99.1 |
| Branch 1 |none | LAN | 192.168.11.0/24 | 192.168.11.1 |
| Branch 2 | none | LAN | 192.168.12.0/24 | 192.168.12.1 |

### Links and servers
| Item | Address |
|---|---|
| HQ-R1 Gi0/1 to B1-R1 Gi0/1 | 10.0.0.1/30 and 10.0.0.2/30 |
| HQ-R1 Gi0/2 to B2-R1 Gi0/1 | 10.0.0.5/30 and 10.0.0.6/30 |
| DNS server | 192.168.99.10 (static) |
| HQ-SW1 management | 192.168.99.2 |

### HQ-SW1 ports
| Port | Connects to | VLAN |
|---|---|---|
| Gi0/1 | HQ-R1 Gi0/0 | Trunk (10, 20, 30, 99) |
| Fa0/1 | Staff PC | 10 |
| Fa0/2 | POS PC | 20 |
| Fa0/3 | Guest PC | 30 |
| Fa0/4 | DNS server | 99 |

### What I configured
- [ ] VLANs 10, 20, 30 and 99 on HQ-SW1, with access ports assigned
- [ ] Trunk between HQ-SW1 and HQ-R1, native VLAN 99
- [ ] Router-on-a-stick sub interfaces on HQ-R1, and a pool n each branch router
- [ ] Static routes between HQ and both branches, and a default route on each branch
- [ ] DNS records on the server (`server.company.local`, `intranet.company.local` )
- [ ] ACL `GUEST_IN`: Guest can only use DNS, and cannot reach Staff, POS, Management or the branches
- [ ] SSH enabled, with Telent disabled
- [ ] Port security on the HQ access ports

### Key Configuration

Full files are in the `config` folder. Examples:
```
! HQ-R1: router-on-a-stick
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

! HQ-R1: static routes to the branches
ip route 192.168.11.0 255.255.255.0 10.0.0.2 
ip route 192.168.12.0 255.255.255.0 10.0.0.6

! HQ-R1: guest ACL
ip access-list extended GUEST_IN
 permit udp 192.168.30.0 0.0.0.255 host 192.168.99.10 eq domain
 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
 deny ip 192.168.30.0 0.0.0.255 192.168.99.0 0.0.0.255
 deny ip 192.168.30.0 0.0.0.255 192.168.11.0 0.0.0.255
 deny ip 192.168.30.0 0.0.0.255 192.168.12.0 0.0.0.255
 permit ip any any
```
## Verification
Tick the result and link a screenshot for each test.
| # | Test | Command | Expected result | Result | Evidence |
|---|---|---|---|---|---|
| 1 | VLANs and ports| `show vlan brief` | fa0/1 in VLAN 10, Fa0/2 in VLAN 20, Fa0/3 in 30, Fa0/4 in 99 | | |
| 2 | Trunk | `show interface trunk` | Gi0/1 trunking. VLANs 10,20,30,99 | | |
| 3 | Router interfaces | `show ip interface brief` | All interfaces up/up with correct addresses | | |
| 4 | DHCP | `ipconfig /all` on each PC | Address from the correct subnet, DNS 192.168.99.10 | | |
| 5 | Between VLANs | Staff to POS: ` ping` | Replies | | |
| 6 | Between sites | HQ PC to Branch 1 PC: `ping`, `tracert` | Replies, path through 10.0.0.x | | |
| 7 | Between branches | Branch 1 PC to Branch 2 PC: `ping` | Replies | | |
| 8 | DNS | `ping server.company.local` | Resolves to 192.168.99.10 | | |
| 9 | Guest blocked | Guest to POS: `ping` | Fails | | |
| 10 |  Guest DNS allowed | Guest: `nslookup server.company.local` | Resolves | | |
| 11 | ACL hits | `show access-lists` | Match counters increase | | |
| 12 | SSH | `ssh -l admin 192.168.99.1` | Login prompt | | |
| 13 | Port security | `show port-security interface f0/1` | Secure, one MAC learned | | |

## Issues I ran into

**Issue 1:** `show interfaces trunk` returned no output.
- Cause: The router interface was off.
- fix: int g0/0 > no shutdown > end

**Issue 2:** PCs in VLAN 20 and 30 got no IP address
- Cause: Ip configuration for ipv4 address was static instead of DHCP
- fix: Switched Ip configuration from static to DHCP


### Files
- `packet-tracer/`: the `-pkt` file
- `configs/`: show running-config` for each router and switch
- `screenshots/`: topology and test evidence

## Related project
[Break-and-Fix-Troubleshooting Portfolio](../break-and-fix-troubleshooting-portfolio): this network with faults introduced and fixed

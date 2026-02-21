# Network Engineer Home Lab – Part 3 (Advanced Enterprise Lab)

---

## You’re going to add:

 HSRP (Gateway Redundancy)

 EtherChannel (Link Aggregation)

 Spanning Tree Optimization

 DMZ Network

 Extended ACL Security Policies

 Site-to-Site VPN (simulation)

 SNMP Monitoring

 Network Performance + Failure Testing
---

# Step 1: Upgrade Topology (High Availability Design)
Add:

* 1 more Layer 3 Switch (Core-2)
* 1 more Access Switch
* 1 DMZ Server

Keep both routers from Part 2

  Architecture
         [Router1] -------- [Router2]
             |                   |
        -------------------------------
        |        CORE LAYER         |
        |   [Core1] ---- [Core2]    |
        -------------------------------
             |            |
         (Access1)    (Access2)
          /   \          /   \
        PCs  PCs      PCs   PCs

                |
               DMZ
            (Web Server)

Now you have:

Redundancy
Load balancing
Failover capability
---
# Step 2: Configure HSRP (Gateway Redundancy)

We’ll make Core1 + Core2 act as a virtual default gateway.

On Core1
```
interface vlan 10
 ip address 192.168.10.2 255.255.255.0
 standby 1 ip 192.168.10.1
 standby 1 priority 110
 standby 1 preempt
```
On Core2
```
interface vlan 10
 ip address 192.168.10.3 255.255.255.0
 standby 1 ip 192.168.10.1
 standby 1 priority 100
 standby 1 preempt
```
Result

Virtual Gateway: 192.168.10.1

Core1 = Active
Core2 = Standby

Test:
Shut Core1 → traffic should continue
---
# Step 3: Configure EtherChannel (Port Aggregation)

Bundle links between switches for speed + redundancy.

On Core Switch
```
interface range g0/1 - 2
 channel-group 1 mode active
```
On Access Switch
```
interface range g0/1 - 2
 channel-group 1 mode active
Configure Port-Channel
interface port-channel 1
 switchport mode trunk
```
Verify
```
show etherchannel summary
```
Now you have:

Increased bandwidth
Fault tolerance
---
# Step 4: Optimize Spanning Tree (STP)

Prevent loops but control which switch is root.

Force Core1 as Root
```
spanning-tree vlan 10 root primary
```
Backup Root (Core2)
```
spanning-tree vlan 10 root secondary
```
Verify
* show spanning-tree
---
# Step 5: Create a DMZ Network

Separate public-facing resources from internal LAN.

DMZ VLAN
```
vlan 50
 name DMZ
```
Core Switch Interface
```
interface vlan 50
 ip address 192.168.50.1 255.255.255.0
```
DMZ Server
IP: 192.168.50.10
GW: 192.168.50.1
---

# Step 6: Secure DMZ with ACLs

Allow:
Internet → DMZ (HTTP/HTTPS)

Block:
DMZ → Internal LAN

ACL Example
```
access-list 110 permit tcp any host 192.168.50.10 eq 80
access-list 110 permit tcp any host 192.168.50.10 eq 443
access-list 110 deny ip 192.168.50.0 0.0.0.255 192.168.0.0 0.0.255.255
access-list 110 permit ip any any
```
Apply:
```
interface vlan 50
 ip access-group 110 in
```
---

# Step 7: Simulate Site-to-Site VPN

On Router1
```
crypto isakmp policy 10
 encryption aes
 hash sha
 authentication pre-share
 group 2

crypto isakmp key cisco address 172.16.0.2
```
IPsec Phase 2
```
crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac

access-list 120 permit ip 192.168.0.0 0.0.255.255 10.10.0.0 0.0.255.255

crypto map VPN-MAP 10 ipsec-isakmp
 set peer 172.16.0.2
 set transform-set VPN-SET
 match address 120

interface g0/1
 crypto map VPN-MAP
```

## Repeat mirrored config on Router2.

---

# Step 8: Configure SNMP Monitoring

On Core Switch:
```
snmp-server community public RO
snmp-server location LAB
snmp-server contact admin@lab.local
```
You can now monitor:

Interface status
CPU usage
Traffic
---

# Step 9: Failure Testing (MOST IMPORTANT)

Simulate real outages:

Pull Links
Shut Core1 → HSRP failover
Break EtherChannel link
Disable trunk

🔥 Observe:
show standby
show etherchannel summary
show spanning-tree

# You now have:

* High Availability Network (HSRP)
* Link Aggregation (EtherChannel)
* Loop Prevention (STP tuning)
* Secure DMZ Architecture
* VPN Connectivity
* Network Monitoring (SNMP)
* Enterprise Redundancy Design

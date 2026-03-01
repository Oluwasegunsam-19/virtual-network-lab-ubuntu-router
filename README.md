# virtual-network-lab-ubuntu-router
3-VM VirtualBox Network Lab with Ubuntu Router, NAT, IP Forwarding and Windows Server Client
                    
                    Internet
                        |
                VirtualBox NAT
                        |
               (enp0s3 - 10.0.2.15)
                 VM1 - Ubuntu Router
               (enp0s8 - 192.168.10.1)
                        |
                 Internal Network (LAN1)
            -------------------------------
            |                             |
      VM2 - Linux Client          Windows Server
      192.168.10.2                192.168.10.3
      GW: 192.168.10.1            GW: 192.168.10.1

**IP Adressing Plan**

| Device         | Interface     | IP Address   | Subnet        | Default Gateway |
| -------------- | ------------- | ------------ | ------------- | --------------- |
| Ubuntu Router  | enp0s3 (NAT)  | 10.0.2.15    | 255.255.255.0 | 10.0.2.2        |
| Ubuntu Router  | enp0s8 (LAN1) | 192.168.10.1 | 255.255.255.0 | —               |
| Linux Client   | LAN1          | 192.168.10.2 | 255.255.255.0 | 192.168.10.1    |
| Windows Server | LAN1          | 192.168.10.3 | 255.255.255.0 | 192.168.10.1    |

**1. IP Forwarding Enabled (Layer 3 Routing)**

sudo sysctl -w net.ipv4.ip_forward=1
Made persistent via /etc/sysctl.conf.

**2. NAT Configuration (Edge Router Functionality)**

sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
sudo iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT
sudo iptables -A FORWARD -i enp0s3 -o enp0s8 -m state --state RELATED,ESTABLISHED -j ACCEPT

_This allowed internal hosts (192.168.10.0/24) to access external networks through address translation.__

**3. Static Gateway Configuration on Clients**

Each internal VM was configured with:

Default Gateway: 192.168.10.1
DNS: 8.8.8.8

_This ensured proper packet forwarding via the Ubuntu router._

**Troubleshooting & Problem Solving**

**Issue 1**: No Internet Connectivity from Clients

Root Cause: NAT forwarding rules missing
Resolution: Applied correct iptables MASQUERADE and FORWARD rules

**Issue 2**: VM1 Could Not Reach Internet

Root Cause: NAT adapter misconfiguration and host firewall interference
Resolution:

Verified ip route

Confirmed default route via 10.0.2.2

Restarted VirtualBox networking

Validated Windows firewall settings

**Issue 3**: Packet Forwarding Disabled

Root Cause: IP forwarding not enabled in kernel
Resolution: Enabled net.ipv4.ip_forward

**Validation & Testing**

Connectivity confirmed via:

ping 192.168.10.1
ping 8.8.8.8
ping google.com


Tests performed from:

Ubuntu Router

Linux Client

Windows Server

All nodes achieved successful outbound connectivity.

**Packet Flow Explanation**

When VM2 accesses the internet:

1. Packet sent to default gateway (192.168.10.1)

2. Ubuntu router forwards packet via enp0s3

3. Source IP translated (MASQUERADE)

4. VirtualBox NAT forwards to host network

5. Response returned and state-tracked back to VM2

This replicates a real-world edge firewall/router deployment model.

# ESXi Setup

## Requirements

- Bare metal with at least 3 NIC*.
- At least 2 public IP address:
  - 1 for VMkernel NIC.
  - 1 for router.
  - Note: check with your provider first, not all provider can provide more than 1 public IP.
- ISOs:
  - pfSense
  - Ubuntu Server
  - Lubuntu

> Some providers use unique mechanism to assign IP address, so you may get away with 2 NIC.

## Installation

1. Install ESXi on bare metal.

2. Access VM management interface on your server via web browser with its public IP. 
   - Initial login: `root/(blank)`

3. Assign free license, get it by registering from VMware (optional).

4. By default, ESXi will already setup 1 virtual switch with 2 port groups:
   - Virtual switch `vSwitch0`.
     - Port group `VM Network`.
     - Port group `Management Network`.
     - Connected to `vmnic0`.

   Create new virtual switch called `WAN`, ensure that it's connected with physical NIC that have your other public IP. Accordingly, create port group `WAN` that is connected to `WAN` virtual switch.

   > In case you only have 2 NIC, the WAN should be `VM Network`. So the above step is not necessary.

   Create new virtual switch called `LAN`, it should be connected to other available physical NIC. (note: the physical NIC doesn't have to connected). Also create port group `LAN` that is connected to `LAN` virtual switch.

5. Create new VM for pfSense:
   - Host: any (e.g. vm-pfsense-1)
   - Guest OS: FreeBSD (64-bit)
   - CPU: 1vCPU
   - RAM: 512MB
   - HDD: 8GB
   - IDE controller: LSI Logic Parallel
   - NIC 1: VMXNET 3 - WAN switch
   - NIC 2: VMXNET 3 - LAN switch

   Start pfSense installation, use guided disk setup and specify no VLAN then install `Open-VM-Tools`.

   > Note: If you use OVH, then you have to assign virtual mac from your failover IP to NIC that is connected with WAN.

6. Configure pfSense via Web.

   The best way is to create a new vm, which boot to lubuntu live cd then use vmrc to connect to that vm.

   Access & configure pfSense via web browser. 
   - Initial login: `admin/pfsense`
   - Host: any (e.g. vm-pfsense-1)

   **WAN**

   - Set IP as your public IP.
   - Set upstream gateway.
   - If you're using OVH:
     - Use failover IP, specify subnet mask 32
     - Set MAC with virtual MAC from failover IP
     - Left upstream gateway empty (for now).
     - Go to `system > routing > gateways` and create new gateway. Use gateway of server IP, make it default, click display advanced and check `use non-local gateway`.
     - Go back to `WAN` and make sure the default gateway is set to our newly-created gateway.

   **LAN**

   - Set IP
   - Enable DHCP

   **Firewall > Rules**

   - Add rule to enable ping
   - Action: Pass
   - Interface: WAN
   - Address: IPv4
   - Protocol: ICMP
   - ICMP Subtypes: any
   - Source: any
   - Dest: any

   **Firewall > NAT > Port Forward**

   - Add port forward to service within LAN

   - Interface: WAN
   - Protocol: TCP
   - Dest Port: HTTP to HTTP
   - Redirect target IP: dest service LAN IP
   - Redirect target Port: dest service LAN port

   - Interface: WAN
   - Protocol: TCP
   - Dest Port: HTTPS to HTTPS
   - Redirect target IP: dest service LAN IP
   - Redirect target Port: dest service LAN port

   **LAN calling WAN address issue**

   - Add host overrides on `Services > DNS Resolver`
   - Modify DNS Servers on `System > General Setup`
   - See https://doc.pfsense.org/index.php/Why_can%27t_I_access_forwarded_ports_on_my_WAN_IP_from_my_LAN/OPTx_networks for more information regarding this issue

## Misc

Just so the answer is somewhere, you can put a nice little script in /usr/local/etc/rc.d like

    #!/bin/sh
    ###route.sh : create route on initialisation
    /sbin/route add -net xxx.xxx.xxx.254/32 -iface emx
    /sbin/route add default xxx.xxx.xxx.254

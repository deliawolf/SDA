# SDA

Cisco SD-Access provides deep visibility into all endpoints on your network, authors access control policies, segments the network, and commands the network to be able to enforce the policies dynamically and automatically.

Some key benefits about the Cisco SD-Access solution are listed below:

1. The fabric enables stretched subnets, which allow an IP subnet to be stretched via the overlay. STP as the solution is routed access and allows Equal-Cost Multipath (ECMP). Therefore, packet forwarding to a single destination can occur over multiple best paths with equal routing priority.

2. The fabric enables local switching of WLAN traffic at the edge node with the help of the anycast gateway.

3. With the Cisco DNA Center acting as a single pane of glass, it allows centralized automation, contextual visibility, and troubleshooting.

4. The fabric enables any service or policy on any port.

5. The fabric enables two levels of segmentation for both LAN and WLAN with the help of a virtual network (or virtual routing and forwarding [VRF]) and scalable group tag (SGT).

6. The fabric enables seamless mobility for users and endpoints and policy is applied end-to-end.

## Cisco SD-Access Fabric

A fabric is an overlay. An overlay network is a logical topology that is used to virtually connect devices and is built on top of some arbitrary physical underlay topology.

### Cisco SD-Access Fabric Terminology
1. Underlay network: The underlay network is defined by physical switches and routers that are part of the campus fabric.
2. Overlay network: An overlay network runs over the underlay to create a virtualized network. Virtual networks isolate both data plane traffic and control plane behavior among the virtualized networks from the underlay network.

### Cisco SD-Access Fabric Roles
The following components make up the Cisco SD-Access solution:

1. Cisco DNA Automation: It provides simple GUI management and intent-based automation (for example, Network Control Platform [NCP]) and context sharing.

2. Cisco DNA Assurance: Data collectors (for example, Neighbor Discovery Protocol [NDP]) analyze endpoint-to-application flows and monitor fabric status

3. Identity Services: Network Admission Control (NAC) and ID systems, for example, Cisco Identity Services Engine (ISE) for dynamic endpoint-to-group mapping and policy definition

4. Control Plane Nodes: Map system that manages endpoint-to-device relationships

5. Fabric Border Nodes: A fabric device (for example, Core) that connects the external Layer 3 networks to the SD-Access fabric

6. Fabric Edge Nodes: A fabric device (for example access or distribution) that connects wired endpoints to the SD-Access fabric

7. Fabric Wireless Controller: A fabric device (wireless LAN controller [WLC]) that connects APs and wireless endpoints to the SD-Access fabric.

## Cisco SD-Access Fabric Components
### Edge Node Role
Edge node provides first-hop services for users or devices connected to a fabric.
### Host Pools
Host pools provide the basic IP functions that are necessary for the attached endpoints:
### Anycast Gateway
Anycast gateway provides a single Layer 3 default gateway for IP-capable endpoints(it works like Redudancy protocol, vrrp,etc)
### Border Node Role
The fabric border nodes serve as the gateway between the Cisco SD-Access fabric site and the networks external to the fabric. The fabric border node is responsible for network virtualization interworking and SGT propagation from the fabric to the rest of the network.
### Control-Plane Node Role
The Cisco SD-Access fabric control plane node is based on the LISP map-server (MS) and map-resolver (MR) functionality combined on the same node. The control plane database tracks all endpoints in the fabric site and associates the endpoints to fabric nodes, decoupling the endpoint IP address or MAC address from the location (closest router) in the network.
#### Cisco SD-Access Key Components
The fabric consists of three key components:

1. Control plane: based on LISP

2. Data plane: based on VXLAN

3. Policy plane: based on Cisco TrustSec

## Cisco SD-Access Fabric Control Plane Based on LISP
LISP is a protocol that enables separation of endpoint identification and its location.
### LISP Operations
When using LISP, the device IP address represents only the device identity. When the device moves, its IP address remains the same in both locations, and only the location ID changes.
### Control-Plane Node Register and Resolution
1. When the endpoints are onboarded on the fabric edge switches, the LISP map-register message is sent from the fabric edge switches to the control plane node. This process helps in creating the host tracking database on the control plane.
2. When a host in the Branch site wants to communicate with 10.2.2.2, the fabric edge connected to the Branch site sends a LISP map-request message to the control plane node, asking for routing locator information for endpoint 10.2.2.2.
3. The control plane node looks into its database and does a proxy-reply to the Branch fabric edge, informing about the routing locator information (2.1.2.1) for the endpoint (10.2.2.2), that it is connected behind.
4. This step completes the control plane query flow and now the Branch fabric edge device knows where to send the traffic to facilitate the communication for the destination endpoint (10.2.2.2).
### Fabric Operation: Fabric Internal Forwarding (Edge-to-Edge)
1. There is a web server hosted within the fabric with a DNS authority record of D.abc.com, which resolves to IP address 10.2.2.2.
2. An endpoint in the Branch site with an IP address of 10.1.0.1 wants to communicate with the web server with IP address 10.2.2.2, and the request hits the fabric edge connected to the Branch site.
3. The fabric edge connected to the Branch site checks its map-cache table and finds the mapping entry for the IP address 10.2.2.2 with a routing locator IP of 2.1.2.1.
4. The fabric edge connected to the Branch encapsulates the packets in a VXLAN header, with the outer IP header having source IP 1.1.1.1 and destination IP 2.1.2.1 and the inner IP header having source IP of 10.1.0.1 and destination IP of 10.2.2.2.
5. The destination fabric edge with IP address of 2.1.2.1 strips off the VXLAN header and forwards the packet to the destination endpoint.
### Fabric Operation: Fabric Internal Forwarding (Border-to-Edge)
1. There is a web server hosted within the fabric with DNS authority record of D.abc.com, which resolves to IP address 10.2.2.2.
2. An endpoint in the internet with a public IP address of 192.3.0.1 wants to communicate with the web server with IP address 10.2.2.2, and the request hits the fabric border.
3. Traffic hits the fabric border that connects the fabric to the outside world, The fabric border checks its map-cache table and finds the mapping entry for the IP address 10.2.2.2 with a routing locator IP of 2.1.2.1.
4. The fabric border encapsulates the packets in the VXLAN header, with the outer IP header having a source IP of 4.4.4.4 and destination IP of 2.1.2.1, and the inner IP header having a source IP of 192.3.0.1 and destination IP of 10.2.2.2.
5. The destination fabric edge with an IP address of 2.1.2.1 strips off the VXLAN header and forwards the packet to the destination endpoint.
### Fabric Operation: Host Mobility-Dynamic EID Migration
1. The host with IP address 10.2.1.10 is connected behind a fabric edge with IP address 12.1.1.1. There are three routing entries created for the host, one for the EID space, one for the /32 prefix local route, and one for the LISP0 interface route.
2. The fabric edge sends a map-request to the control plane to update the host tracking database for the host with IP address 10.2.1.10/32, informing routing locator IP address (switch ip) as 12.1.1.1.
3. When a host wants to communicate to a DC server in subnet 10.10.10.0/24, the fabric edge sends a request to the control plane to resolve the routing locator, which is 12.0.0.1.
4. The host moves to another location and now gets connected behind the fabric edge with IP address 12.2.2.1. The fabric edge installs routes in its table.
5. The fabric edge updates the control plane about the endpoint and the host tracking database gets updated with the routing locator IP of the new fabric edge, where the host is now connected.
6. The control plane updates the fabric edge with the IP address 12.1.1.1 to clear the LISP entry for the endpoint.
7. Traffic flow for the host to DC server is now facilitated by the fabric edge with the IP address 12.2.2.1.

## Cisco SD-Access Fabric Data Plane Based on VXLAN
The Cisco SD-Access fabric data plane is based on VXLAN. VXLAN supports both Layer 2 and Layer 3 overlay. It preserves the original Ethernet header. RFC 7348 defines the use of VXLAN as a way to overlay a Layer 2 network on top of a Layer 3 network.
## Cisco SD-Access Fabric Policy Plane Based on Cisco TrustSec
The Cisco SD-Access fabric policy plane is based on Cisco TrustSec.

The VXLAN header carries the fields for VRF and SGT that are being used in network segmentation and security policies
1. The first level of segmentation is at the VRF (virtual network) level.
2. The second level of segmentation is at the SGT level.

## Cisco SD-Access Wireless Architecture
Cisco SD-Access wireless is defined as the integration of wireless access in the SD-Access architecture to gain all the advantages of fabric and Cisco DNA Center automation.
### Cisco SD-Access Wireless Architecture: Simplifying Policy and Segmentation
1. AP removes the 802.11 header.
2. AP adds the 802.3/VXLAN/underlay IP header.
3. The AP embeds the policy information in the VXLAN header and forwards it. The client VRF is represented by the Layer 2 virtual network (Layer 2 VXLAN network identifier [VNID]). This Layer 2 VNID is provided to the AP by the WLC upon endpoint authentication.
4. Fast Ethernet de-encapsulates the VXLAN header, looks at the Layer 2 VNID, and maps it to the VLAN and Layer 2 LISP instance. Then Fast Ethernet A does the lookup and encapsulates to the destination Fast Ethernet B
5. Like any other wired fabric traffic, the destination switch applies the policy end-to-end by stripping the VXLAN header and looking in the VNID and SGT values.
### Cisco SD-Access Wireless Basic Workflow: Add WLC to Fabric
1. In Cisco DNA Center, first provision and then add WLC to the fabric domain.
2. Fabric configuration is pushed to the WLC and the WLC is configured with credentials to establish a secure connection to the control plane device.
### Cisco SD-Access Wireless Basic Workflow: AP Join Process
1. User configures AP pool in Cisco DNA Center. Cisco DNA Center preprovisions a configuration macro on the Fast Ethernets.
2. AP is plugged in and powers up. Fast Ethernet discovers it is an AP via Cisco Discovery Protocol and applies the macro/autoconf to assign the switch port to the right VLAN. This is only true if no authentication is configured on the switch port where the AP is connected.
3. AP gets an IP address via DHCP in the overlay. Next, Fast Ethernet registers the AP as a “special” wired host into the fabric.
4. The fabric edge registers the IP address (EID) of the AP and updates the control plane.
5. The AP learns and joins the WLC using traditional methods. The fabric AP joins as a local mode AP (no support for Flex or WLC over WAN). The latency between the AP and WLC is less than 20 ms.
6. The WLC checks if it is fabric-capable (Wave 2 or Wave 1 APs).
7. If the AP is supported for fabric, the WLC queries the control plane to know if the AP is connected to the fabric.
8. The control plane replies to the WLC with the RLOC. This means the AP is attached to the fabric and will be shown as “fabric-enabled.”
9. The WLC does a Layer 2 LISP registration for the AP in the control plane (also called AP “special” secure client registration), which is used to pass important metadata information from the WLC to the Fast Ethernet.
10. In response to this proxy registration, the control plane notifies the fabric edge and passes the metadata received from the WLC (flag that says it is an AP and the AP IP address).
11. The fabric edge processes the information, learns it is an AP, and creates a VXLAN tunnel interface to the specified IP (optimally, the switch side is ready for clients to join).
### Cisco SD-Access Wireless Basic Workflow: Client Onboarding
1. The client authenticates to a fabric-enabled WLAN. The WLC gets SGT from ISE, updates the AP with client Layer 2 VNID and SGT. The WLC knows the RLOC of the AP from the internal database.
2. The WLC proxy registers client Layer 2 information in the control plane. This is the LISP modified message to pass additional information, like the client SGT.
3. Fast Ethernet gets notified by the control plane and adds the client MAC address in the Layer 2 forwarding table and fetches the policy from ISE based on the client SGT.
4. The client initiates a DHCP request.
5. The AP encapsulates it in VXLAN with Layer 2 VNID information.
6. The fabric edge maps the Layer 2 VNID to the VLAN interface and forwards it to DHCP in the overlay (same as for a wired fabric client).
7. The client receives an IP address from DHCP.
8. DHCP snooping (or ARP for static) triggers the client EID registration by the fabric edge to the control plane. This completes the client onboarding process.
### Cisco SD-Access Wireless Basic Workflow: Client Roaming
1. The client roams to AP2 on Fast Ethernet 2 (interswitch roaming). The WLC is notified by the AP.
2. The WLC updates the forwarding table on the AP with client information (SGT, RLOC).
3. The WLC updates the Layer 2 MAC entry in the control plane with the new RLOC Fast Ethernet2.
4. The control plane then notifies the following:
   - The fabric edge Fast Ethernet2 (“roam-to” switch) to add the client MAC address to the forwarding table pointing to the tunnel
   - The fabric edge Fast Ethernet1 (“roam-from” switch) to do clean up for the wireless client
   - The fabric border to update the internal RLOC for this client
5. Fast Ethernet will update the Layer 3 entry (IP) in the control plane database upon receiving traffic.
The roam is Layer 2 because Fast Ethernet2 has the same VLAN interface (anycast gateway).

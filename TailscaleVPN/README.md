Tailscale VPN Deployment & Zero-Trust Network Setup

- Author: Ty Iorfino
- Date: September - November 2025
- Tools Used: Tailscale, Proxmox VE, Pi-hole, Docker, Ubuntu Server, Windows 10/11, MagicDNS, WireGuard, ACL Policies

---

Project Overview
- This project documents how I installed, configured, and deployed Tailscale VPN across a hybrid environment consisting of Proxmox nodes, Linux containers (LXC), Windows hosts, and an IOS system.
- The goal was to create a secure, encrypted, zero-trust mesh network allowing me to connect remotely to my home lab and manage local services safely from anywhere.
- All communication between devices occurs over WireGuard-based encrypted tunnels, ensuring privacy and integrity while allowing remote access.

---

System Architecture
- Environment: 
  - Proxmox VE hosting multiple Linux Containers (LXC) and Dockerized services
  - Windows 11 workstation for client connectivity and administrative access
  - Pi-hole container acting as a DNS sinkhole and network exit node for ad blocking and traffic routing
  - Tailscale used to interconnect all nodes under a single identity and enforce zero-trust policies
- Core Concepts Used:
  - MagicDNS for hostname-based communication between nodes
  - Access Control Lists to manage least-privilege access
  - Exit Nodes for encrypted remote browsing through my home network
  - Subnet Routing for LXC and Docker access within Proxmox

--- 

Technology Used
- Tailscale:	VPN and Zero-Trust network orchestration
- WireGuard:	Encryption backbone for mesh connectivity
- Proxmox VE:	Virtualization platform for hosting containers and VMs
- Pi-hole:	DNS sinkhole and network-level ad-blocker
- MagicDNS:	Hostname resolution and internal name mapping
- ACL Policies:	Access control and least-privilege enforcement
- Docker:	Service containerization
- Ubuntu Server / Windows 11	Operating systems for deployment

---

Deployment steps
1. Installing Tailscale
- I made an account on tailscales offical website and started generating auth keys
<img width="1505" height="692" alt="Screenshot 2025-10-21 165808" src="https://github.com/user-attachments/assets/115d0245-0672-40fc-b5ec-71f7144807cd" />
- I began by installing Tailscale on each system in my network, by going to add device and then picking the OS system of the device
- Based on the operating system picked youll be prompted to install tailscale or run the install script
    - On Ubuntu / LXC containers, I generated an install script that looked like:
        curl -fsSL https://tailscale.com/install.sh | sh
        tailscale up --authkey <your-auth-key> --hostname <device-name>
<img width="1287" height="1262" alt="Screenshot 2025-10-21 165839" src="https://github.com/user-attachments/assets/78e103f6-89e8-4f70-95ed-ed04d424ec4d" />
<img width="1002" height="129" alt="Screenshot 2025-10-21 180425" src="https://github.com/user-attachments/assets/86c08c47-3904-4f76-ae3c-5f1a5a2f42d7" />
    - On Windows:
        I added the device by having tailscale send a connection link to my email
        I then downloaded tailscale on the device and connected it to my personal tailnet
<img width="1896" height="766" alt="Screenshot 2025-10-21 170303" src="https://github.com/user-attachments/assets/d0516e52-918e-4135-a14c-236bf92fd304" />

---

2. Establishing a Secure Mesh Network
- Once all systems were connected, Tailscale automatically formed a peer-to-peer mesh network using the WireGuard protocol.
- This allowed every device to communicate directly and securely, even across NAT or different subnets, without manual port forwarding.
<img width="214" height="504" alt="Screenshot 2025-10-21 170536" src="https://github.com/user-attachments/assets/e9149d84-e5ec-4768-9aaa-0bdfc5b120ca" />

--- 

3. Enabling MagicDNS
- I enabled MagicDNS through the Tailscale Admin Console for automatic hostname resolution:
    - Eliminated the need to remember IPs
    - Allowed access using simple hostnames like proxmox.local or pihole.local
    - Ensured cross-device consistency for DNS queries
<img width="748" height="207" alt="Screenshot 2025-10-27 091718" src="https://github.com/user-attachments/assets/27a2f3be-4046-4dc8-8bb6-05c5a5ced7e5" />

---

4. Configuring Pi-hole as Exit Node
- My Pi-hole container was configured to act as an exit node, routing all outbound internet traffic through my home network:
    sudo tailscale up --exit-node=true --exit-node-allow-lan-access
- This setup ensured:
    - Encrypted browsing from remote locations
    - DNS-level ad and tracker blocking via Pi-hole
    - Consistent network policies across devices
<img width="361" height="95" alt="Screenshot 2025-10-27 091743" src="https://github.com/user-attachments/assets/e2757782-d21b-4737-a077-448dd0c64249" />
<img width="989" height="221" alt="Screenshot 2025-10-22 210219" src="https://github.com/user-attachments/assets/3b73925e-1ef7-464e-b1e9-5455c57f19c8" />
<img width="756" height="169" alt="Screenshot 2025-10-27 091934" src="https://github.com/user-attachments/assets/64c9798c-2e41-46af-8ed9-a7524ca4fb20" />


---

5. Implementing Zero-Trust with ACLs
- Using the Tailscale Admin Console, I defined ACL rules in JSON to control access between users and devices.
For example, admin could access all ports and services while jellyfin could only use ports 8096,8920 and members could access:

"grants": [
		// Admins: anywhere, any port
		{
			"src": ["group:admin"],
			"dst": ["*"],
			"ip":  ["*"],
		},
		// Members: Jellyfin web only
		{
			"src": ["group:member"],
			"dst": ["tag:jellyfin"],
			"ip":  ["*:8096", "*:8920"],

This ensured all users operated under least privilege and only had access to necessary resources.
<img width="1184" height="297" alt="Screenshot 2025-10-27 092911" src="https://github.com/user-attachments/assets/d9a27557-7a44-4b4e-acb1-ed5db04ccdac" />
<img width="1222" height="565" alt="Screenshot 2025-10-27 092943" src="https://github.com/user-attachments/assets/3fdd7965-7f63-406e-9980-83a9bba113a8" />

---

6. Integrating with Proxmox Networking
- I configured Proxmox virtual networks so that containerized services (LXCs and Docker containers) were reachable through the Tailscale interface.
- This allowed remote SSH, web UI, and API access through secure tunnels without public IP exposure.
- This can be tested by end-to-end testing using commands like ping.

---

7. Admin Console & Key Management
- I managed all devices and users through the Tailscale Admin Console:
- Reviewed connected devices and activity
- Rotated authentication keys and made sure tailscale service was updated
- Verified key expiration and session validity
- Monitored ACL policies 
<img width="742" height="605" alt="Screenshot 2025-10-27 094604" src="https://github.com/user-attachments/assets/454fe66c-8ba4-4045-b6bc-c081dd6577aa" />
  - When one of my keys expire I just run: "Sudo tailscale up" and then it gives me a link to reauth the device
  - To force a key reset I have to revoke the key manually in the GUI and then generate a new key and apply it
  
---

8. End-to-End Testing
- To validate network integrity, I conducted connectivity and DNS propagation tests across:
  - Ubuntu LXCs
  - Docker containers
  - Windows hosts
- Commands like ping and curl were used to verify encrypted tunnels and hostname resolution via MagicDNS.
<img width="646" height="178" alt="Screenshot 2025-10-27 143315" src="https://github.com/user-attachments/assets/8e1aa2cd-677f-443d-855c-28ebfdbf4e77" />
<img width="693" height="259" alt="Screenshot 2025-10-27 143126" src="https://github.com/user-attachments/assets/0f907daa-0be7-4e41-89a6-1a6d47c66b15" />

---

Security Outcomes
- All VPN connections are encrypted end-to-end using WireGuard.
- DNS-level ad-blocking through Pi-hole enhances privacy. 
- Zero-Trust ACL framework enforces least privilege.
- MagicDNS ensures reliable hostname resolution across all devices.
- Remote management of Proxmox and local services without exposing public ports.

---

Issues Faced
- After the download process on my LXC container services like pihole, jellyfin, and vaultwarden I noticed the tailscale up command wouldnt work and the dashboard wouldnt show the device
<img width="1005" height="149" alt="Screenshot 2025-10-21 180508" src="https://github.com/user-attachments/assets/15674bdc-be9c-4385-9459-5118e4da1adf" />

- To fix this issue due to the service being an LXC you need to open the LXC config file while on proxmox root and add:
<img width="648" height="65" alt="Screenshot 2025-10-21 181155" src="https://github.com/user-attachments/assets/44d3814d-a4c9-475e-93ab-237617522297" />
  - nano /etx/pve/lxc/<pctID>.conf # to enter the LXC config file

- After adding these lines you need to start and then stop the LXC container so the changes will take place, then run tailscale up, you should get a direct link to tailscale to connect the device, this allows the LXC to connect to the TUN
<img width="321" height="88" alt="Screenshot 2025-10-21 181603" src="https://github.com/user-attachments/assets/a1fdd233-1280-496b-b0f6-e96fee297226" />
<img width="361" height="115" alt="Screenshot 2025-10-21 181657" src="https://github.com/user-attachments/assets/dc43652f-16dc-49bc-a664-743c397f702a" />

---

Conclusion
- Through this project, I deployed a fully encrypted, zero-trust VPN infrastructure that connects multiple environments under one secure network.
- This setup demonstrates my ability to deploy and manage modern VPN solutions using Tailscale and WireGuard, while integrating with virtualization platforms like Proxmox and enhancing privacy with Pi-hole and using security postures like a zero-trust framwork. 

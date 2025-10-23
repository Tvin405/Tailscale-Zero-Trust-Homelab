🔒 Tailscale VPN Deployment & Zero-Trust Network Setup

Author: Ty Iorfino
Date: October 2025
Tools Used: Tailscale, Proxmox VE, Pi-hole, Docker, Ubuntu Server, Windows 10/11, MagicDNS, WireGuard, ACL Policies

📘 Project Overview

This project documents how I installed, configured, and deployed Tailscale VPN across a hybrid environment consisting of Proxmox nodes, Linux containers (LXC), and Windows hosts.
The goal was to create a secure, encrypted, zero-trust mesh network allowing me to connect remotely to my home lab and manage local services safely from anywhere.

All communication between devices occurs over WireGuard-based encrypted tunnels, ensuring privacy and integrity while simplifying remote access.

🧱 System Architecture

Environment:

Proxmox VE hosting multiple Linux Containers (LXC) and Dockerized services

Windows 11 workstation for client connectivity and administrative access

Pi-hole container acting as a DNS sinkhole and network exit node for ad blocking and traffic routing

Tailscale used to interconnect all nodes under a single identity and enforce zero-trust policies

Core Concepts Used:

MagicDNS for hostname-based communication between nodes

ACLs (Access Control Lists) to manage least-privilege access

Exit Nodes for encrypted remote browsing through my home network

Subnet Routing for LXC and Docker access within Proxmox

⚙️ Step-by-Step Configuration
1. Installing Tailscale

I began by installing Tailscale on each system in my network:

On Ubuntu / LXC containers:

curl -fsSL https://tailscale.com/install.sh | sh
tailscale up --authkey <your-auth-key> --hostname <device-name>


On Windows:

Downloaded the Tailscale client from tailscale.com/download
.

Signed in using my Tailscale account (GitHub/OAuth).

Connected the device to my personal tailnet.

2. Establishing a Secure Mesh Network

Once all systems were connected, Tailscale automatically formed a peer-to-peer mesh network using the WireGuard protocol.
This allowed every device to communicate directly and securely, even across NAT or different subnets, without manual port forwarding.

3. Enabling MagicDNS

I enabled MagicDNS through the Tailscale Admin Console for automatic hostname resolution:

Eliminated the need to remember IPs

Allowed access using simple hostnames like proxmox.local or pihole.local

Ensured cross-device consistency for DNS queries

4. Configuring Pi-hole as Exit Node

My Pi-hole container was configured to act as an exit node, routing all outbound internet traffic through my home network:

sudo tailscale up --exit-node=true --exit-node-allow-lan-access


This setup ensured:

Encrypted browsing from remote locations

DNS-level ad and tracker blocking via Pi-hole

Consistent network policies across devices

5. Implementing Zero-Trust with ACLs

Using the Tailscale Admin Console, I defined ACL rules in JSON to control access between users and devices.
For example, administrative nodes were restricted to specific authenticated users only:

{
  "ACLs": [
    {
      "Action": "accept",
      "Users": ["ty@domain.com"],
      "Ports": ["proxmox:22", "pihole:80"]
    }
  ]
}


This ensured all users operated under least privilege and only had access to necessary resources.

6. Integrating with Proxmox Networking

I configured Proxmox virtual networks so that containerized services (LXCs and Docker containers) were reachable through the Tailscale interface.
This allowed remote SSH, web UI, and API access through secure tunnels without public IP exposure.

7. Admin Console & Key Management

I managed all devices and users through the Tailscale Admin Console:

Reviewed connected devices and activity

Rotated authentication keys for security hygiene

Verified key expiration and session validity

Monitored traffic routes and ACL policy application

8. End-to-End Testing

To validate network integrity, I conducted connectivity and DNS propagation tests across:

Ubuntu LXCs

Docker containers

Windows hosts

Commands like ping, dig, and curl were used to verify encrypted tunnels and hostname resolution via MagicDNS.

🔐 Security Outcomes

✅ All VPN connections are encrypted end-to-end using WireGuard.
✅ DNS-level ad-blocking through Pi-hole enhances privacy and reduces bandwidth waste.
✅ Zero-Trust ACL framework enforces least privilege and limits lateral movement.
✅ MagicDNS ensures reliable hostname resolution across all devices.
✅ Remote management of Proxmox and local services without exposing public ports.

🧰 Technologies Used
Tool	Purpose
Tailscale	VPN and Zero-Trust network orchestration
WireGuard	Encryption backbone for mesh connectivity
Proxmox VE	Virtualization platform for hosting containers and VMs
Pi-hole	DNS sinkhole and network-level ad-blocker
MagicDNS	Hostname resolution and internal name mapping
ACL Policies	Access control and least-privilege enforcement
Docker	Service containerization
Ubuntu Server / Windows 11	Operating systems for deployment
📄 Conclusion

Through this project, I built a fully encrypted, zero-trust VPN infrastructure that connects multiple environments under one secure network.
This setup demonstrates my ability to design, deploy, and manage modern VPN solutions using Tailscale and WireGuard, while integrating with virtualization platforms like Proxmox and enhancing privacy with Pi-hole.

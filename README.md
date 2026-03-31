# Network-Traffic-Analysis-Lab

A hands-on lab environment designed to simulate, capture, and defend against network reconnaissance attacks.

## Objective
The goal of this project evolved from observing basic port scans (Nmap) in Wireshark to implementing active defense mechanisms (UFW/iptables) and finally deploying a full-stack SIEM (Security Information and Event Management) for automated threat detection.

---

## Lab Setup
- **Hypervisor:** VirtualBox 7.1.4
- **Attacker:** Kali Linux (IP: `10.0.2.4`)
- **Victim/Security Server:** Ubuntu Desktop (IP: `10.0.2.15`)
- **Security Stack:** Wazuh Manager and Agent, Wireshare, UFW, iptables
- **Network Mode:** NAT Network

### Challenges Overcome: Network Configuration
1. **Issue:** During the initial setup, the VMs were unable to communicate despite being on the default NAT setting. 
- **Solution:** I created a dedicated **NAT Network** in VirtualBox and enabled **Promiscuous Mode**. This ensured both VMs were on the same `10.0.2.x` subnet and allowed Wireshark to "sniff" the traffic between them.

2. **Issue:** During the Wazuh installation, the Ubuntu VM ran out of storage due to the Indexer's requirements.
- **Solution:** Performed a manual disk expansion in VirtualBox and utilized GParted to resize the Linux partition and claim unallocated space, ensuring a stable environment for the SIEM database. Used several commands to restart the process and reinstall the Wazuh manager.
---

## Phase 1: Attack & Capture
I simulated a Stealth attack using Kali Linux to identify open ports on the Ubuntu target.

### Steps Taken:

### 1. Attacker View:
The Nmap scan successfully identified open ports and services on the target machine.
![Nmap Scan Results](https://github.com/user-attachments/assets/7a3226a1-2572-4666-8d09-cfed047117d9)

Command:
sudo nmap -sS -T4 10.0.2.15

### 2. Captured Evidence:
The capture shows the "Half-Open" SYN packets being sent to the target.
![Wireshark Nmap Capture](https://github.com/user-attachments/assets/98dd19f5-2cec-43df-b584-08a8b168ddd8)

### 3. Analysis:
The capture shows a "Half-Open" connection attempt. Kali sends a `SYN` packet, Ubuntu responds with `SYN, ACK` (for open ports), and Kali immediately sends `RST` to close the connection before it's fully established. This avoids traditional logging on the victim machine.

## Phase 2: Hardening & Defense Implementation
After analyzing the attack, I implemented defensive measures to secure the Ubuntu target.

### Evidence:
![Hardened Scan Results](https://github.com/user-attachments/assets/05388e30-07b5-4ae6-9ccf-3ebc6049ab20)

### Defensive Actions Taken:
1. **Firewall Activation (UFW):**
   I enabled the Uncomplicated Firewall to set a "Deny by Default" policy.
Command:
sudo ufw enable
sudo ufw default deny incoming

2. **Rate Limiting (iptables):**
To mitigate automated scanning, I implemented a rule to limit new TCP SYN packets to 1 per second.
Command:
sudo iptables -I INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j RETURN
sudo iptables -I INPUT -p tcp --syn -j DROP


## Phase 3: SIEM Implementation (Wazuh)
To upgrade the lab to industry standards, I deployed Wazuh, an open-source security monitoring platform, to automate threat identification.

- **Manager and Agent:** Installed the Wazuh Manager as the central "Brain" and configured a local Agent on the Ubuntu endpoint to monitor system integrity and network events.
- **Real-Time Alerting:** Re-running the Nmap scan now triggers high-severity alerts on the Wazuh Dashboard, providing instant visibility without manual packet sniffing.


## Conclusion & Key Learnings
This lab demonstrated the critical difference between **passive monitoring** and **active defense**. 

- **Visibility is Key:** Without Wireshark, the stealth SYN scan would have gone completely unnoticed by the standard OS logs.
- **Dropping vs. Rejecting:** By configuring the firewall to `DROP` packets rather than `REJECT` them, I effectively "cloaked" the machine. This forces attackers to wait for timeouts, significantly slowing down their reconnaissance phase.
- **Defense in Depth:** Combining a static firewall (UFW) with dynamic rate-limiting (iptables) provides a much more robust security posture than either tool alone.

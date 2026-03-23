# Network-Traffic-Analysis-Lab

A hands-on lab environment designed to simulate, capture, and defend against network reconnaissance attacks.

## Objective
The goal of this project is to observe how a port scan (Nmap) appears to a defender using **Wireshark** and to implement active defense mechanisms using **UFW** and **iptables**.

---

## Lab Setup
- **Hypervisor:** VirtualBox 7.x
- **Attacker:** Kali Linux (IP: `10.0.2.4`)
- **Victim:** Ubuntu Desktop (IP: `10.0.2.15`)
- **Network Mode:** NAT Network

## Lab Setup
* **Hypervisor:** VirtualBox
* **Attacker:** Kali Linux
* **Victim:** Ubuntu Desktop

### Challenges Overcome: Network Configuration
During the initial setup, the VMs were unable to communicate despite being on the default NAT setting. 
- **Issue:** Kali was only assigned an IPv6 address, and Ubuntu was on a different subnet, preventing a "Ping" connection.
- **Solution:** I created a dedicated **NAT Network** in VirtualBox and enabled **Promiscuous Mode**. This ensured both VMs were on the same `10.0.2.x` subnet and allowed Wireshark to "sniff" the traffic between them.

---

## Phase 1: Attack & Capture
I simulated a reconnaissance attack using Kali Linux to identify open ports on the Ubuntu target.

### Steps Taken:
1. Started a live capture in Wireshark on the Ubuntu machine.
2. Executed a TCP SYN Stealth Scan from Kali: `sudo nmap -sS -T4 10.0.2.15`.
3. Analyzed the resulting packet flow.

### Captured Evidence:
![Wireshark Nmap Capture](<img width="1271" height="805" alt="nmap_wireshark_capture" src="https://github.com/user-attachments/assets/98dd19f5-2cec-43df-b584-08a8b168ddd8" />
)

### Analysis:
The capture shows a "Half-Open" connection attempt. Kali sends a `SYN` packet, Ubuntu responds with `SYN, ACK` (for open ports), and Kali immediately sends `RST` to close the connection before it's fully established. This avoids traditional logging on the victim machine.

## 🛡️ Phase 2: Hardening & Defense Implementation
After analyzing the attack, I implemented defensive measures to secure the Ubuntu target.
### Evidence:
![Hardened Scan Results](<img width="1274" height="815" alt="nmap_hardened_capture" src="https://github.com/user-attachments/assets/05388e30-07b5-4ae6-9ccf-3ebc6049ab20" />
)

### Defensive Actions Taken:
1. **Firewall Activation (UFW):**
   I enabled the Uncomplicated Firewall to set a "Deny by Default" policy.
Command:
  sudo ufw enable
  sudo ufw default deny incoming

3. Rate Limiting (iptables):
To mitigate automated scanning, I implemented a rule to limit new TCP SYN packets to 1 per second.
Command: 
sudo iptables -I INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j RETURN
sudo iptables -I INPUT -p tcp --syn -j DROP

## 🎓 Conclusion & Key Learnings
This lab demonstrated the critical difference between **passive monitoring** and **active defense**. 

- **Visibility is Key:** Without Wireshark, the stealth SYN scan would have gone completely unnoticed by the standard OS logs.
- **Dropping vs. Rejecting:** By configuring the firewall to `DROP` packets rather than `REJECT` them, I effectively "cloaked" the machine. This forces attackers to wait for timeouts, significantly slowing down their reconnaissance phase.
- **Defense in Depth:** Combining a static firewall (UFW) with dynamic rate-limiting (iptables) provides a much more robust security posture than either tool alone.

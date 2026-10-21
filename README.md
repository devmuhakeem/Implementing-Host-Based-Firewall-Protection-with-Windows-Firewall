# Implementing Host-Based Firewall Protection with Windows Firewall

A hands-on lab blocking Remote Desktop and FTP access to a Windows server using host-based firewall rules, confirming beforehand that both services were reachable, then systematically blocking and verifying each one.

## Scenario
A web server (10.10.1.16) was reachable via both RDP and FTP from another machine on the network with no host-level restriction in place. The task was to lock down both services using Windows Firewall's inbound rules, discovering along the way that neither service maps to a single rule.

## What I did

### 1. Confirmed the exposure
- Connected to the web server via Remote Desktop Connection from an admin machine — succeeded without resistance, confirming no host-based restriction existed yet

### 2. Blocked RDP — three separate rules, not one
Via Control Panel → System and Security → Windows Firewall → Advanced Settings → Inbound Rules, set **Block the connection** on:
- **Remote Desktop - Shadow (TCP-In)**
- **Remote Desktop - User Mode (TCP-In)**
- **Remote Desktop - User Mode (UDP-In)**

### 3. Verified the RDP block
- Attempted the same RDP connection from the admin machine — failed with a connection error, confirming all three rules together closed the path

### 4. Set up the FTP test
- Temporarily disabled Windows Firewall entirely on the web server
- Connected via `ftp 10.10.1.16` from the admin machine with admin credentials — succeeded, confirming the FTP service itself was reachable before re-enabling protection

### 5. Blocked FTP — again, three separate rules
Re-enabled the firewall with recommended settings, then blocked:
- **FTP Server (FTP Traffic-In)**
- **FTP Server Passive (FTP Passive Traffic-IN)**
- **FTP Server Secure (FTP SSL Traffic-In)**

### 6. Verified the FTP block
- Attempted `ftp 10.10.1.16` again from the admin machine — got a connection timeout, confirming all three FTP-related rules together blocked the service

## Key takeaways
- A "service" as a user understands it (RDP, FTP) is often represented as multiple distinct rules at the firewall level, covering different modes (standard vs. shadow RDP), transports (TCP vs. UDP), or variants (plain, passive, and SSL/TLS-secured FTP)
- Blocking only the most obviously-named rule can still leave a working path in through a mode or transport you didn't account for
- Set rules to explicitly **Block** rather than just disabling them — Block states intent clearly; disabling just stops evaluation and can leave a more permissive default to take over
- Always verify a block from the other side of the connection, not by reading the firewall's rule list — a rule that looks right in the GUI and a rule that actually stops the traffic are two different claims
- Host-based firewalls are a defense-in-depth layer, not a substitute for network segmentation — they hold even when something upstream is misconfigured or absent

## Tools
Windows Firewall (with Advanced Security), Remote Desktop Connection, FTP client, Windows Server

---
*Completed in an isolated training lab environment against provisioned Windows VMs.*

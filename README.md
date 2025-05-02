# intrustion-detection-system-scripts
The purpose of the scripts in this repository is to use the "Zeek" Intrusion Detection System (IDS) in order to detect and label different attack types. Currently I am building a testbed made to simulate these types of traffic:
1. Benign (Normal) Web Traffic
2. Hostile Traffic. These will consist of the following Cyber Attacks:
   A. Reconnaissance & Scanning
     I. Port Scans (TCP, UDP)
     II. OS Fingerprinting
     III. Host Discovery
     IV. Network Probing
   B. DoS & DDoS
     I. UDP, ICMP, TCP SYN, PSH ACK Floods
     II. SlowLoris, Amplification, SSDP, MQTT Publish Floods
     III. Reflective Attacks (DNS, ICMP, UDP, TCP)
   C. Brute Force
     I. SSH, FTP, Telnet, RTSP, MQTT Brute Force
     II. Hydra, Sparta, Dictionary Tasks
   D. MITM, Spoofing, & ARP Poisoning
     I. ARP Spoofing
     II. DNS Spoofing
     III. Man-in-the-Middle Attempts
   E. Botnet & Malware Behavior
     I. Mirai, BASHLITE, Torii, Okiru Traffic Patterns
     II. Known C2 IPs/Domains
     III. Unusual Beaconing or Payloads
   F. Command & Control
     I. File Upload & Download
     II. Reverse Shells, Heartbeat Communications
   G. Credential Access
     I. SSL Renegotiation Abuse
     II. ARP/DNS Spoofing for Credential Theft
     III. MQTT Message Interception
   H. Privelage Escalation & Exploit Injection Scripts
     I. SQL Injection, Web Exploit Attempts (Tomcat, Video Injection)
     II. Remote to Local Exploits (R2L), Zero-Day Signs
   I. May create sub-modules or enhance detections for:
     I. Worm Propagation Detection
     II. Keylogging Detection (Via unusual reverse traffic or beaconing)
     III. Ransomeware Detection (Rapid file access logs or command bursts)

These detection scripts are described below:

## Reconnaissance & Scanning
**Script Name:** scan_detect.zeek
**Typical Alerts:** Scan::Port_Scan notices
**What it Watches:** Connection Attempts in conn.log
**How it Decides Something is Bad:** Counts how many unique ports/hosts a single IP touches in a given time window. If the count crosses a certain threshold, it flags a Port/Host Scan.

## DoS & DDoS
**Script Name:** dos_ddos_detect.zeek
**Typical Alerts:** DDoS::Flood notices
**What it Watches:** Raw packet flow and TCP flags
**How it Decides Something is Bad:** Counts total packets and SYNs every minute. If a source exceeds a packet-rate or SYN-rate threshold, it raises a DoS/DDoS Flood alert.

## Brute Force
**Script Name:** brute_force_detect.zeek
**Typical Alerts:** Bruteforce::Guessing notices
**What it Watches:** Failed auth events in ssh.log, ftp.log, optional MQTT hooks
**How it Decides Something is Bad:** Tracks failed logins per IP/user pair in a given time window, >= a certain number of failures willl raise a Brute-Force Guessing Notice then reset the counter.

## MITM, Spoofing, & ARP Poisoning
**Script Name:** spoof_mitm_detect.zeek
**Typical Alerts:** MITM::ARPSpoof notices
**What it Watches:** ARP traffic in arp.log
**How it Decides Something is Bad:** Keeps a MAC-address cache for every IP. If the same IP claims a different MAC later, it will raise an ARP-Spoof/MITM flag.

## Botnet & Malware Behavior
**Script Name:** botnet_behavior_detect.zeek
**Typical Alerts:** Botnet::Payload notices
**What it Watches:** Telnet scans & malicious HTTP fetches
**How it Decides Something is Bad:** Waatches fr Mirai-style Telnet probes on ports 23/2323 and for Bashlite payload URLs that match a BusyBox + wget/tftp pattern, then raises Botnet Payload/Scan alerts.

## Command & Control
**Script Name:** c2_traffic_detect.zeek
**Typical Alerts:** C2::Beacon notices
**What it Watches:** Outbound HTTP request timing
**How it Decides Something is Bad:** Records time between a host's HTTP requests. >= a certain threshold with gaps < a certain time threshold will mark it as a periodic Beaconing/C2 Traffic.

## Credential Access
**Script Name:** cred_access_detect.zeek
**Typical Alerts:** Cred::WeakSSL (plus any MQTT notices you add)
**What it Watches:** SSL handshakes & MQTT connects
**How it Decides Something is Bad:** Flags weak SSL versions and leaves hooks for counting MQTT auth failure or message interception. Primary goal is Credential-Theft Indicators.

## Privelage Escalation & Exploit Injection Scripts
**Script Name:** exploit_detect.zeek
**Typical Alerts:** Exploit::SQLi, Exploit::XSS notices
**What it Watches:** HTTP URIs
**How it Decides Something is Bad:** Simple regex match for SQL-injection or XSS patterns in incoming requests. Reports Web Exploit Attempts.

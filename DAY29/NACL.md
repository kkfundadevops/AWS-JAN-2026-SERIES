# AWS Network ACL (NACL) Complete Documentation

## 1. What is a Network ACL

A Network ACL, also called NACL, is a security layer attached at the **subnet level** in AWS.

It controls which traffic is allowed to enter or leave a subnet.

You can think of it like this:

* **VPC** = gated community
* **Subnet** = a street inside that community
* **NACL** = security check at the street entrance
* **EC2 instance** = a house inside the street

So before traffic reaches an instance, subnet-level rules can already allow or block it.

---

## 2. Main purpose of NACL

NACL is used to control traffic at subnet level for:

* additional security filtering
* allowing only required ports
* blocking specific traffic
* controlling inbound and outbound access
* providing a subnet-wide security boundary

It acts as an extra protection layer in addition to Security Groups.

---

## 3. Important characteristics of NACL

### 1) NACL supports both Allow and Deny rules

This is one of the most important points.

With NACL, you can:

* explicitly allow traffic
* explicitly deny traffic

This is useful when you want to block a certain IP range, port, or protocol.

### 2) NACL is stateless

This is the most important interview point.

Stateless means:

if you allow inbound traffic, the return traffic is **not automatically allowed**.

You must separately create rules for:

* inbound traffic
* outbound traffic

Both directions must be open properly.

### 3) Rules are processed in number order

Each NACL rule has a rule number such as:

* 100
* 110
* 120

AWS checks rules from lowest number to highest number.

### 4) Lowest rule number gets highest priority

Example:

* Rule 100 = Deny port 80
* Rule 200 = Allow port 80

Result:
Rule 100 is checked first, so traffic is denied.

### 5) First matching rule takes effect

As soon as a packet matches a rule, AWS stops checking the remaining rules.

### 6) NACL is associated with a subnet

A NACL does not attach directly to EC2 instance.

It attaches to the subnet, so all resources inside that subnet are affected by that NACL.

### 7) One subnet can have only one NACL at a time

But one NACL can be associated with multiple subnets.

---

## 4. What is Inbound traffic

Inbound means traffic coming **into the subnet**.

Examples:

* Internet user opening your website
* traffic from another subnet entering your subnet
* traffic from on-premises network entering your subnet
* traffic from VPN or Direct Connect entering your subnet

Simple meaning:

Traffic is entering your subnet from outside.

---

## 5. What is Outbound traffic

Outbound means traffic going **out of the subnet**.

Examples:

* EC2 instance sending response back to user
* EC2 instance accessing Google or software repository
* EC2 instance downloading packages from internet
* traffic moving from your subnet to another subnet
* traffic going from AWS to on-premises

Simple meaning:

Traffic is leaving your subnet and going outside.

---

## 6. Understanding Stateless behavior with example

Suppose a client on the internet wants to access your web server.

Client sends request to server on:

* destination port 80

The server processes the request and sends response back.

In NACL, this return traffic is not automatically allowed.

So you must configure:

### Inbound

Allow traffic coming to server on port 80

### Outbound

Allow return traffic to client on ephemeral ports

If outbound rule is not present, response will fail.

This is why NACL is called stateless.

---

## 7. What are Ephemeral Ports

Ephemeral ports are temporary ports used for return traffic or client-side communication.

When a client connects to a server:

* server listens on fixed port like 80 or 443
* client uses a random temporary port
* server sends response back to that temporary port

These temporary ports are called ephemeral ports.

### Common range

Generally:

* 1024 to 65535

In many AWS teaching examples, this full range is used for simplicity.

### Example

User opens website:

* Client source port = 51025
* Server destination port = 80

Response comes back:

* Server source port = 80
* Client destination port = 51025

So to allow reply traffic, NACL must allow outbound traffic to ephemeral port range.

---

## 8. Correct traffic flow example

### Scenario

A user accesses a web server running on EC2 in a public subnet.

### Step 1

Client sends request from random source port, for example 51025

### Step 2

Request reaches server on destination port 80

### Step 3

Web server processes request

### Step 4

Server sends response back to client’s temporary source port 51025

### NACL requirement

#### Inbound NACL

Allow:

* source: internet client IP
* destination port: 80

#### Outbound NACL

Allow:

* destination: internet client IP
* destination port: ephemeral range 1024–65535

Without outbound ephemeral port rule, response may not go back properly.

---

## 9. Default NACL

Every VPC comes with a **default NACL**.

### Default NACL behavior

It allows all inbound and all outbound traffic by default.

So it is open unless you modify it.

This is why many people say default NACL does not block anything initially.

### Important point

Default NACL already has allow rules, so traffic works immediately unless changed.

---

## 10. Custom NACL

When you create a new NACL manually, it behaves differently from default NACL.

### New custom NACL behavior

Initially, it denies all traffic until you add rules.

That means:

* no inbound allowed
* no outbound allowed

You must manually create all required allow rules.

This is an important practical point because people create a new NACL and then wonder why traffic stops.

---

## 11. NACL rule structure

Each NACL rule contains:

* Rule number
* Type (HTTP, HTTPS, SSH, Custom TCP, etc.)
* Protocol
* Port range
* Source or destination
* Allow / Deny

Example inbound rule:

* Rule number: 100
* Protocol: TCP
* Port: 80
* Source: 0.0.0.0/0
* Action: Allow

Example outbound rule:

* Rule number: 100
* Protocol: TCP
* Port: 1024-65535
* Destination: 0.0.0.0/0
* Action: Allow

---

## 12. How rule priority works

Suppose you have these inbound rules:

* Rule 100: Deny traffic from 192.168.1.0/24
* Rule 200: Allow HTTP from 0.0.0.0/0

If a packet comes from 192.168.1.10 on port 80:

* Rule 100 matches first
* traffic is denied
* Rule 200 is never checked

So always remember:

**lowest rule number wins because it is evaluated first**

---

## 13. NACL vs Security Group

This is one of the most important comparison topics.

# Security Group

* works at instance level
* supports only allow rules
* stateful
* return traffic is automatically allowed
* easier to manage for EC2 protection

# NACL

* works at subnet level
* supports both allow and deny
* stateless
* inbound and outbound both must be configured
* useful for subnet-wide filtering

---

## 14. Stateful vs Stateless in simple language

### Security Group = Stateful

If request comes in and is allowed, response goes out automatically.

You usually do not need to separately worry about return path.

### NACL = Stateless

If request comes in and is allowed, response is not automatically allowed.

You must explicitly allow both directions.

---

## 15. Practical example: Web server in public subnet

Suppose you host a website on EC2.

You want users from internet to access it.

### Required inbound NACL rules

* Allow HTTP 80 from 0.0.0.0/0
* Allow HTTPS 443 from 0.0.0.0/0

### Required outbound NACL rules

* Allow ephemeral ports 1024–65535 to 0.0.0.0/0

Why?
Because clients connect from random source ports and expect responses on those ports.

---

## 16. Practical example: SSH access to EC2

Suppose admin wants to SSH into EC2.

### Inbound NACL

* Allow TCP 22 from admin public IP

### Outbound NACL

* Allow ephemeral ports 1024–65535 back to admin IP

If outbound ephemeral ports are not allowed, SSH session may fail or not work properly.

---

## 17. Practical example: Private subnet accessing internet via NAT Gateway

Suppose an EC2 instance in private subnet wants to install packages.

Traffic flow:

* EC2 sends request to internet through NAT Gateway
* internet sends response back
* subnet NACL must allow the traffic both ways

So NACL rules must support:

* outbound to required destination ports like 80/443
* inbound return traffic through ephemeral ports

---

## 18. Real-time use cases of NACL

NACL is commonly used for:

### 1) Blocking a suspicious IP range

You can deny a malicious CIDR block directly at subnet level.

### 2) Additional subnet-wide protection

Even if Security Group is open, NACL can provide one more filtering layer.

### 3) Public subnet control

You can tightly control which ports are exposed to internet.

### 4) Segmentation

You can apply different subnet-level rules for public, private, and database subnets.

### 5) Compliance and governance

Some organizations want subnet-level deny rules as part of their security policy.

---

## 19. When to use Security Group and when to use NACL

### Use Security Group for

* regular EC2 protection
* application port access
* instance-level security
* day-to-day access management

### Use NACL for

* subnet-level broad protection
* deny rules
* IP range blocking
* extra security layer

In real projects, both are often used together.

---

## 20. Important default deny concept

At the end of every NACL, there is an implicit deny.

That means if traffic does not match any allow rule, it is denied automatically.

So if you forgot to add a required rule, traffic will be blocked.

This is why rule planning is very important.

---

## 21. Common mistake in NACL configuration

### Mistake 1

Only inbound rule added, outbound forgotten

Result:
Request reaches server, but response fails

### Mistake 2

Port 80 allowed, but ephemeral ports not allowed

Result:
Website may not load properly due to response blocking

### Mistake 3

Wrong rule order

Result:
Deny rule with lower number blocks traffic before allow rule

### Mistake 4

Custom NACL created but no rules added

Result:
Everything stops working

### Mistake 5

Confusing NACL with Security Group

Result:
People expect automatic return traffic, but NACL does not work that way

---

## 22. Example rules for public subnet web server

### Inbound rules

* Rule 100: Allow TCP 80 from 0.0.0.0/0
* Rule 110: Allow TCP 443 from 0.0.0.0/0
* Rule 120: Allow TCP 22 from your-office-public-ip/32

### Outbound rules

* Rule 100: Allow TCP 1024-65535 to 0.0.0.0/0
* Rule 110: Allow TCP 80 to 0.0.0.0/0
* Rule 120: Allow TCP 443 to 0.0.0.0/0

This kind of setup is usually enough for demo environment.

---

## 23. Example rules for database subnet

If database should only accept traffic from app subnet:

### Inbound

* Allow DB port like 3306 only from app subnet CIDR

### Outbound

* Allow ephemeral ports back to app subnet CIDR

This helps keep database subnet restricted.

---

## 24. How to explain Subnet in simple words

A subnet is a smaller network inside a VPC.

If VPC is a big colony, subnet is one street or one block.

Resources like EC2, RDS, NAT Gateway, etc. are placed inside subnets.

You can design subnets for different purposes:

* public subnet
* private subnet
* database subnet

Each subnet can have its own route table and NACL association.

---

## 25. Easy analogy for classroom

### VPC

A big apartment community

### Subnet

One street or one block in that community

### NACL

Security guard checking who can enter or leave the street

### Security Group

Security lock on each individual house

This explanation is simple and students understand quickly.

---

## 26. NACL and Route Table difference

Many students confuse them.

### Route Table

Decides where traffic should go

### NACL

Decides whether traffic should be allowed or blocked

Simple line:

* Route table = path selection
* NACL = permission checking

Both are needed.

---

## 27. NACL and Security Group together

Traffic to EC2 is generally affected by both:

1. Route table sends traffic correctly
2. NACL checks subnet-level permission
3. Security Group checks instance-level permission

If any one layer blocks the traffic, final communication fails.

So troubleshooting should always check:

* route table
* NACL
* security group
* OS firewall
* application port

---

## 28. Interview-ready points

These are the important points students should remember:

* NACL works at subnet level
* Security Group works at instance level
* NACL is stateless
* Security Group is stateful
* NACL supports allow and deny
* Security Group supports only allow
* lowest rule number has highest priority
* first matching rule takes effect
* default NACL allows all traffic
* custom NACL denies all traffic until rules are added
* ephemeral ports are important for return traffic
* NACL is used as an additional security layer

---

## 29. Short explanation for students

Network ACL is a subnet-level firewall in AWS. It controls traffic entering and leaving the subnet. It is stateless, which means inbound and outbound rules must both be configured. It supports both allow and deny rules. Rules are evaluated in number order, and the lowest rule number has the highest priority. Default NACL allows all traffic, but a new custom NACL blocks everything until rules are added.

---

## 30. Short explanation of Ephemeral ports for students

Ephemeral ports are temporary ports used during communication. When a client sends a request to a server on port 80 or 443, the response usually goes back to the client’s temporary port, not to the same server listening port. That is why NACL outbound or inbound return path rules must often allow ephemeral port range.

---

## 31. Trainer note: one important correction

One line often gets explained incorrectly:

“Server replies back on port 56789”

Better explanation is:

The **client** usually opens a random temporary source port like 56789.
The server listens on port 80.
When the server responds, it sends traffic **from port 80 to client port 56789**.

This is the clearer and technically correct way to explain it.

---

## 32. Final summary

Network ACL is subnet-level protection in AWS.
It is useful for controlling traffic entering and leaving subnets.
Unlike Security Groups, it is stateless, so both inbound and outbound rules must be opened properly.
It supports allow and deny, which makes it powerful for broader subnet filtering.
Rule order is very important because the lowest numbered matching rule is applied first.
Default NACL allows all traffic, while a new custom NACL blocks everything until rules are added.

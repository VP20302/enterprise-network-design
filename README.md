Enterprise Network Design: Cisco On-Premise vs. AWS Cloud

A comparative network infrastructure design for a mid-sized FMCG distributor in Regina, Saskatchewan. The project delivers two complete, costed solutions - a Cisco on-premise network and an AWS Virtual Private Cloud deployment - evaluates them across cost, scalability, security, and performance, and recommends a phased hybrid architecture.

Course: CNET123 - Saskatchewan Polytechnic, Business Information Systems

Deliverable: Full technical design document, IP addressing plan, AWS build, 10-year TCO analysis, and client presentation

The Client Problem

A fast-moving consumer goods distributor operating from a single Regina headquarters needed a secure, scalable network to support 1,258 employees across nine departments, with 10 Gb performance and shared printer allocation.

Department sizing, including projected growth:

Customer Service - 800 employees, 20% growth, 960 total
Sales - 200 employees, 10% growth, 220 total
Quality Control - 100 employees, 10% growth, 110 total
HR - 50 employees, no growth projected
Marketing - 40 employees, no growth projected
Technical - 30 employees, no growth projected
Financial - 20 employees, no growth projected
Strategy - 10 employees, no growth projected
Legal - 8 employees, no growth projected

Requirements: department-level isolation, capacity for growth, and continuous uptime with minimal single points of failure. Printer allocation at one per 30 employees produced a requirement of 52 networked printers, bringing total port demand to 1,523.

Solution 1: Cisco On-Premise Network
Topology

A two-tier physical architecture with a three-tier logical model:

Access layer - star topology. 35 x Catalyst 9200-48P switches, each department on dedicated switches with no cross-department sharing. Larger departments (Customer Service, Sales, Quality Control) receive multiple switches; every access switch carries redundant uplinks to both core switches.
Core layer - partial mesh. 2 x Catalyst 9300-48P handling all inter-VLAN routing and inter-department traffic, with redundant links eliminating single points of failure.
Perimeter layer (logical). 2 x Catalyst 8300 edge routers for WAN and internet connectivity, plus 4 x Firepower 1120 next-generation firewalls deployed in active-passive pairs - two at the border, two internal to segment sensitive departments.

Star at the access layer localizes faults and simplifies troubleshooting; partial mesh at the core provides redundancy and optimal inter-department paths without full-mesh complexity.

IP Addressing and Subnetting

Class B - 172.16.0.0/16, subnetted with VLSM.

Class A (10.0.0.0/8) would waste 16 million addresses. Class C (192.168.0.0/24) caps at 254 hosts per subnet - insufficient for Customer Service's 960+ hosts. Class B provides 65,534 addresses with room to right-size every department.

Subnet allocation, listed as department, subnet mask, usable hosts, and network ID:

Customer Service - /22, 1,022 usable hosts - 172.16.4.0/22
Sales - /24, 254 usable hosts - 172.16.1.0/24
Quality Control - /25, 126 usable hosts - 172.16.10.0/25
Marketing - /26, 62 usable hosts - 172.16.0.0/26
HR - /26, 62 usable hosts - 172.16.2.0/26
Technical - /27, 30 usable hosts - 172.16.3.0/27
Financial - /27, 30 usable hosts - 172.16.8.0/27
Strategy - /28, 14 usable hosts - 172.16.9.0/28
Legal - /28, 14 usable hosts - 172.16.9.16/28

Why VLSM over fixed-length subnetting: a uniform mask large enough for Customer Service would have wasted over 1,000 addresses on Legal alone. VLSM sizes each subnet to the nearest power of two above its requirement, leaving contiguous gaps between subnets so departments can expand without renumbering. All ranges are non-overlapping, RFC 1918 compliant, and isolate broadcast domains at each department boundary.

Solution 2: AWS Virtual Private Cloud

FMCG-VPC - 10.0.0.0/16, deployed across two availability zones (ca-central-1a, ca-central-1b) for high availability.

14 subnets - two public (Public-A, Public-B) and the remainder private, one per department
Internet Gateway (FMCG-IGW) attached to the VPC for public subnet connectivity
NAT Gateway (FMCG-NAT) in a public subnet with an attached Elastic IP
Split route tables - the public table routes 0.0.0.0/0 to the Internet Gateway; the private table routes 0.0.0.0/0 to the NAT Gateway

This gives private subnets outbound internet access for updates and API calls while blocking all unsolicited inbound traffic - security and functionality without exposing internal systems.

10-Year Cost Analysis
Cisco On-Premise: $343,211.90 total
Initial hardware - $135,364.75
Lifecycle replacement (Years 5, 6, 9, 10) - $135,364.75
Setup labour (80 hours at $60/hr) - $4,800.00
Maintenance (5% of hardware cost annually) - $67,682.40

Hardware breakdown: 35 x Catalyst 9200-48P ($106,226.75), 2 x Catalyst 9300-48P ($7,568.00), 2 x Catalyst 8300 ($13,770.00), 4 x Firepower 1120 ($7,800.00).

AWS Cloud: $4,482.00 total
VPC, subnets, and route tables - free
Internet Gateway - free
Elastic IP (attached to a running resource) - free
NAT Gateway hourly charge - 87,600 hours at $0.045/hr = $3,942.00
NAT data processing - 12,000 GB at $0.045/GB = $540.00

The AWS figure represents baseline networking only - compute, storage, and higher data volumes would raise it substantially. The comparison isolates network infrastructure cost.

Comparative Findings

Cost. On-premise requires $135,364.75 upfront as capital expenditure; AWS requires nothing upfront and bills as operating expense. Over ten years the on-premise network totals $343,211.90 against $4,482.00 for baseline AWS networking, though AWS costs scale with usage while on-premise costs stay largely fixed.

Scalability. Expanding the on-premise network means procuring and installing hardware, a process measured in weeks, and often means over-provisioning against uncertain growth. AWS provisions new subnets and capacity in minutes and supports multi-region expansion without new physical sites.

Security. On-premise gives full physical control with VLAN segmentation and ACLs enforcing department isolation, which simplifies compliance for regulated data. AWS provides built-in DDoS protection, IAM, and encryption under a shared responsibility model, but security depends on correct configuration of security groups and NACLs.

Performance and reliability. The on-premise 10 Gb LAN delivers low-latency internal traffic with no internet dependency, which matters for large file transfers and real-time applications. AWS offers a 99.99% multi-AZ SLA and automated disaster recovery, but internal traffic becomes subject to ISP reliability and internet latency.

Recommendation: Phased Hybrid Architecture

Neither solution alone fits the client. A full cloud migration introduces latency into daily internal workflows; a pure on-premise build forces over-provisioning against uncertain growth.

Keep on-premise: Sales, Customer Service, and Quality Control operations requiring low-latency 10 Gb LAN, plus Finance, Legal, and HR data where physical control simplifies compliance.

Move to AWS: backup and disaster recovery, remote workforce access, dev/test environments, and burst capacity for seasonal demand.

Implementation Roadmap
Year 0-1 - Deploy Cisco core infrastructure; migrate print servers and internal databases on-premise.
Year 2-3 - Implement AWS Storage Gateway for backups; establish AWS Direct Connect for secure cloud links.
Year 5+ - Shift dev/test environments to AWS; deploy AWS Workspaces for remote teams.

This sequencing avoids all-or-nothing migration risk and preserves the option to rebalance workloads in either direction as costs and requirements evolve.

Skills Demonstrated

Networking: VLSM subnetting, IP address planning, VLAN segmentation, star and partial mesh topology design, inter-VLAN routing, redundant link design, firewall placement and defense-in-depth

Cloud: AWS VPC architecture, public/private subnet design, Internet Gateway and NAT Gateway configuration, route table management, multi-AZ deployment

Analysis: Requirements gathering, hardware lifecycle planning, 10-year total cost of ownership modelling, comparative evaluation, technical recommendation and roadmap development

Documentation: Technical specification writing, network design documentation, cost justification, stakeholder presentation

Team and Contributions

Group project completed by Gabriel Dannang, Vic Novelozo, John Aaron Elorde, Vandit Parikh, and Muhammad Khan.

My contributions (Vandit Parikh): the Client Requirements Analysis and the complete Cisco Network Design - departmental subnet and printer requirements, topology selection and justification, IPv4 address class selection, the full VLSM subnetting plan, network device selection, and the 10-year on-premise cost estimation.

Teammate contributions: AWS VPC architecture and cloud cost estimation (John Aaron Elorde); comparative analysis and final recommendation (Vic Novelozo); introduction (Muhammad Khan).

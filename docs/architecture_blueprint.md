PART 2 — Cloud, Security & IoT Deployment Blueprint 

# Cloud, Security & IoT Deployment Blueprint
## Task 9: Choose a Distributed Architecture and Communication Plan
### 1. Architectural Choice: Hybrid Model
The Smart City Network Operations platform selects a **Hybrid Distributed Architecture** to coordinate the three zone controllers with the central Smart City Operations dashboard.
- **Transparency:** The central dashboard interacts with the cluster using unified high-level REST and MQTT topics, disguising internal peer-to-peer state synchronization across zone controllers from the operator interface.
- **Fault Tolerance:** If the central dashboard node fails, individual zone controllers continue running local sensor task dispatch uninterrupted using the Part 1 scheduler and Banker's-Algorithm engine.
- **Scalability:** High-frequency local task execution is handled regionally within each zone, preventing centralized bandwidth bottlenecks as additional sensor controllers join the network.
- **Single Point of Failure Elimination:** Peer-to-peer ledger replication among zone controllers ensures that critical state metadata (e.g., global compute credit allocations) remains accessible even during central node connectivity drops.
### 2. Communication Data Flows
- **(a) Real-time Public-Safety Alert Push:**
  - **Type:** Asynchronous.
  - **Protocol:** **MQTT** (Message Queuing Telemetry Transport over TLS).
  - **Reasoning:** Public-safety alerts must be dispatched non-blockingly to ensure edge processing loops are not delayed. MQTT's minimal header overhead and publish-subscribe model provide sub-second delivery with QoS 1 guaranteed delivery.
- **(b) Daily Sensor Log Archival Upload:**
  - **Type:** Asynchronous.
  - **Protocol:** **HTTPS** (REST API with Chunked Multipart Upload).
  - **Reasoning:** Archival uploads consist of large bulk datasets. Asynchronous HTTPS allows reliable transmission, retry handling, and standard TLS encryption for archival storage in cloud storage buckets.
---
## Task 10: Design a VPC-Based Network Boundary for the Three Zones
### 1. VPC and Subnet Isolation Structure
The platform utilizes a **Single Virtual Private Cloud (VPC) with 3 Private Subnets** (`Subnet-Zone-A`, `Subnet-Zone-B`, `Subnet-Zone-C`) and **1 Public Subnet** (hosting the API Gateway and Dashboard Application).
### 2. Cross-Zone Isolation Control
To prevent Zone-B resources from directly accessing Zone-A resources:
- **Enforcing Network Control:** **Network Access Control List (NACL) Rules** applied directly at the subnet boundaries of `Subnet-Zone-A`.
- **Rule Configuration:** An inbound rule on `Subnet-Zone-A` explicitly blocks all IPv4 traffic originating from `Subnet-Zone-B`'s CIDR block (`10.0.2.0/24 -> DENY`).
- *Note:* The dashboard application operates downstream of this boundary; the subnet NACL enforces structural network-level isolation across zones.
---
## Task 11: Map a Control to Every Network-Security Objective
1. **Protect Sensitive Data:** Implement **AES-256 Storage Encryption** for local database disks and **TLS 1.3 Encryption in Transit** for sensor transmissions.
2. **Authentication:** Enforce **Mutual TLS (mTLS) with X.509 Digital Certificates** on each zone controller to verify node identity before granting network access.
3. **Authorization:** Implement **Role-Based Access Control (RBAC)** using signed JWT OAuth 2.0 tokens to restrict administrative capabilities according to the least privilege principle.
4. **Prevent Cyber Attacks:** Deploy a **Web Application Firewall (WAF) and Intrusion Prevention System (IPS)** at the VPC ingress boundary to filter malformed payloads and SQL injection attempts.
5. **Secure Communication:** Establish **IPsec VPN Tunnels** for all communications passing between physical zone edge hardware and VPC cloud resources.
6. **Ensure Availability:** Deploy **Multi-AZ Auto Scaling Groups** for compute engines alongside automated Cloudflare DDoS Protection to mitigate volumetric traffic floods.
---
## Task 12: Build the IAM Table and Data-Protection Map
### 1. IAM Role Table

| Role Name | Granted Permission Set | Target Resource Scope |
| :--- | :--- | :--- |
| **Zone Operator** | Read/Execute local scheduler tasks, view local job queues, submit job overrides | Local Zone Controller |
| **City Dashboard Admin** | Full Administrative Access (Read/Write/Delete/Configure System Parameters) | Global Cloud Dashboard |
| **Auditor** | Read-Only access to audit logs, compliance reports, and system telemetry metrics | Global Storage Archives |

### 2. Data State Protection Map
- **Data at Rest:** Encrypted via **AES-256 Bit Encryption (LUKS / dm-crypt)** applied to local disk volumes holding the static `JOBS` list and system logs.
- **Data in Transit:** Secured using **TLS 1.3 Protocol Encryption** for public-safety alert streams transmitted from zone controllers to the central dashboard.
- **Data in Use:** Isolated via **Process Memory Isolation and Hardware Enclaves (AWS Nitro Enclaves)** during execution of the Banker's Algorithm safety check in RAM.
---
## Task 13: Plan IoT Connectivity and Map IoT Architecture Layers
### 1. Sensor/Device Types and Connectivity Selection
1. **Traffic-Camera Trigger:** Connected via **5G Cellular Network**. Provides high bandwidth and ultra-low latency (<5ms) necessary for real-time video stream processing and rapid alert generation.
2. **Environmental Sensor:** Connected via **LoRaWAN**. Offers long-range transmission capability (up to 15 km) with low power consumption suitable for battery-operated environmental monitors.
3. **Wearable Public-Safety Device:** Connected via **NB-IoT (Narrowband IoT)**. Provides deep building signal penetration and optimized energy consumption for personal panic alert triggers.
### 2. 6-Layer IoT Architecture Mapping
1. **Physical Environment Layer:** Physical roadways, traffic intersections, and city environmental monitoring sites.
2. **Perception/Device Layer:** Traffic cameras, air quality monitors, and wearable distress devices.
3. **Gateway Layer:** Regional edge gateways that collect raw sensor signals and convert protocols into standardized JSON packages.
4. **Network Communication Layer:** Cellular 5G, LoRaWAN towers, and IPsec VPN infrastructure routing packets into the VPC network.
5. **Cloud Platform Layer:** The core **Python Job-Scheduler, Peterson's Mutual Exclusion Sync, and Banker's Algorithm Safety Engine from Part 1**.
6. **Application Layer:** The central Smart City Operations Dashboard displaying system metrics, maps, and real-time incident reports.
---
## Task 14: Identify Threats and Mitigations
1. **Threat 1: Man-in-the-Middle (MitM) Packet Sniffing & Command Injection**
   - *Mitigation:* Enforce mandatory **mTLS certificate pinning** across all inter-zone communication channels.
2. **Threat 2: Credential Theft and Unauthorized API Execution**
   - *Mitigation:* Require short-lived **OAuth 2.0 JWT tokens** combined with Multi-Factor Authentication (MFA) for administrative roles.
3. **Threat 3: Distributed Denial of Service (DDoS) on Ingress Endpoints**
   - *Mitigation:* Implement **VPC Ingress Rate-Limiting rules** and cloud-native volumetric DDoS protection (e.g., AWS Shield / Cloudflare).
  
  Task 8: Justify Deployment Choice
Chosen Production Scheduling Family: Non-Preemptive Priority Scheduling with Aging
Selected Family: Priority Scheduling with Dynamic Aging is chosen for production deployment across the zone controllers.
Cited Reasons Rejecting Alternative Families:
Rejection of FCFS: FCFS exhibits a severe convoy effect on this dataset, producing the highest average waiting time of 13.12 ticks and an average turnaround time of 18.62 ticks, forcing critical low-burst safety tasks to wait behind heavy processing jobs.
Rejection of SJF / SRTF Family: Although SRTF achieved the lowest average waiting time (6.00 ticks), both SJF and SRTF require exact advance knowledge of CPU burst times (8, 4, 9, 5, \dots), which cannot be known beforehand for dynamic IoT sensor workloads.
Rejection of Round Robin Family: Round Robin introduces excessive context switching overhead. At quantum 3, the engine recorded 16 context switches across 17 dispatch slices, causing CPU cycles to be lost to OS switching overhead rather than processing real sensor data.

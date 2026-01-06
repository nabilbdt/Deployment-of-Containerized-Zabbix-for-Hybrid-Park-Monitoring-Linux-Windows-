# Centralized Cloud Monitoring Infrastructure on AWS
## Deployment of Containerized Zabbix for Hybrid Park Monitoring (Linux & Windows)

---

### Project Overview
This project demonstrates the implementation of a centralized supervision infrastructure using Zabbix 7.4.6 deployed via Docker. The setup monitors a hybrid environment consisting of Ubuntu Linux and Windows Server instances hosted on AWS.

---
### Infrastructure Architecture

The following diagram illustrates the logical layout of the monitoring environment within AWS, showing the relationship between the containerized Zabbix stack and the hybrid client fleet.

```mermaid
graph TD
    subgraph AWS_Cloud [AWS Cloud Region]
        subgraph VPC [VPC: VPC-Zabbix-Lab]
            subgraph Subnet [Public Subnet]
                
                subgraph Server_Node [EC2: Zabbix Server Host]
                    Z_Server[Zabbix Server Container]
                    Z_Web[Zabbix Web UI Nginx]
                    Z_DB[MySQL 8.0 Database]
                end

                L_Client[EC2: Linux Client]
                W_Client[EC2: Windows Client]

                %% Monitoring Flux
                Z_Server -.->|Port 10050| L_Client
                Z_Server -.->|Port 10050| W_Client
                L_Client -.->|Port 10051| Z_Server
                W_Client -.->|Port 10051| Z_Server
            end
        end
    end

    %% Component Styles
    style AWS_Cloud fill:#f9f9f9,stroke:#333,stroke-width:2px
    style VPC fill:#f0f7ff,stroke:#007acc
    style Server_Node fill:#fff9f0,stroke:#d4a017
    style L_Client fill:#f0fff0,stroke:#2e7d32
    style W_Client fill:#f0f0ff,stroke:#1a237e
```
---

### Infrastructure Architecture
The core environment is built upon a dedicated AWS network to ensure isolated and secure monitoring traffic.

* **VPC**: VPC-Zabbix-Lab (CIDR: 10.0.0.0/16).
* **Zabbix Server**: Ubuntu EC2 instance hosting a Dockerized stack (Zabbix Server, Web Interface, and MySQL).
* **Monitored Clients**: 
    * Ubuntu Linux EC2 Instance.
    * Windows Server EC2 Instance.

<img width="1910" height="610" alt="Screenshot 2026-01-05 165829" src="https://github.com/user-attachments/assets/2f159b5a-0943-4254-8c1c-fd7d7135ecf8" />

---

### Security Configuration
The infrastructure utilizes three distinct AWS Security Groups to enforce the principle of least privilege:

#### 1. Zabbix Server (SG-Zabbix-Server)
Designed to handle incoming web traffic and agent data.
* **Port 80 (HTTP)**: Web UI access.
* **Port 10051**: Zabbix Trapper for active checks and proxies.
* **Port 22 (SSH)**: Administrative access.

<img width="1528" height="468" alt="Security Group Server 1" src="https://github.com/user-attachments/assets/6032b52d-fa02-4d7a-8406-dcf20e070fa0" />
<img width="1910" height="610" alt="Security Group Server 2" src="https://github.com/user-attachments/assets/6ed31b4d-ef1c-494b-b575-89ef7d527e51" />

#### 2. Linux Client (SG-Zabbix-Linux)
* **Port 10050**: Zabbix Agent communication.
* **Port 22**: SSH access.

#### 3. Windows Client (SG-Zabbix-Windows)
* **Port 10050**: Zabbix Agent communication.
* **Port 3389**: Remote Desktop Protocol (RDP) access.

---

### Deployment Steps

#### Step 1: Server Preparation
The Ubuntu host was prepared by installing Docker and Docker Compose to support the containerized application stack.

#### Step 2: Containerized Stack Launch
The core services were deployed using a `docker-compose.yml` file, including:
* **Database**: MySQL 8.0.
* **Server**: Zabbix Server with MySQL support.
* **Web**: Zabbix Web interface backed by Nginx.

```bash
docker-compose up -d
```
#### Step 3: Agent Configuration
Zabbix Agents were deployed on target machines to collect system metrics.
**Linux**: Installed via the standard apt repository.
**Windows**: Deployed via the Zabbix MSI Agent 2 package.
**Network Setup**: All agents were configured to communicate via the Zabbix Server's Private IP (10.0.9.100) to maintain internal traffic integrity.

Monitoring Results
The final implementation provides a unified view of the infrastructure health, verifying active communication across the internal AWS network.
**CPU Utilization**: Real-time performance tracking.
**Memory Usage**: Visualized through centralized dashboard widgets.
**Availability**: Simultaneous monitoring of the Linux Client, Windows Client, and the Zabbix Server itself.

<img width="1919" height="914" alt="Final Monitoring Dashboard" src="https://github.com/user-attachments/assets/3182b8fc-7804-4f0f-b2cc-06f97a3e4a94" />

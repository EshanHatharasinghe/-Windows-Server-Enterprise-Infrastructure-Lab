# -Windows-Server-Enterprise-Infrastructure-Lab
An industrial-grade Windows Server 2022 infrastructure lab designed and implemented using Virtualization. This repository documents the deployment of core identity services, role separation concepts, centralized storage, network management, and security policies following enterprise IT standards.

### 1. Conceptual Enterprise Architecture
In a production enterprise environment, workloads are isolated across dedicated Virtual Machines to guarantee high availability, fault tolerance, and security (Separation of Roles). 

┌────────────────────────────────────────────────────────────────────────┐
│                        Physical Hyper-V / VMware Host                  │
│  ┌───────────────────────┐  ┌───────────────────────┐  ┌────────────┐  │
│  │         DC01          │  │         FS01          │  │   SQL01    │  │
│  │  - Active Directory   │  │  - File & Storage     │  │  - Database│  │
│  │  - DNS / DHCP         │  │  - Shared Folders     │  │  - Engine  │  │
│  └───────────┬───────────┘  └───────────┬───────────┘  └─────┬──────┘  │
└──────────────┼──────────────────────────┼────────────────────┼─────────┘
               │                          │                    │
               └──────────────────┬───────┴────────────────────┘
                                  │ (Internal Virtual Switch)
                                  ▼
                     [ Client Workstation (CL01) ]

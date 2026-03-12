# Hybrid Identity Lab — Microsoft Entra Cloud Sync

## Overview

This lab extends the on-premises Active Directory environment built in the 
microsoft-lab by integrating it with Microsoft Entra ID using the Microsoft 
Entra Provisioning Agent (Cloud Sync). The goal was to simulate a hybrid 
identity environment where user accounts created in on-premises Active 
Directory are automatically synchronised to the cloud.

This mirrors a real-world MSP scenario where organisations maintain on-premises 
AD for legacy systems while leveraging Entra ID for cloud access and modern 
authentication.

---

## Lab Architecture

| Component | Details |
|---|---|
| On-premises domain | lab.local |
| Domain Controller | DC01 (Windows Server 2022) |
| Virtualisation | Oracle VirtualBox |
| Cloud tenant | kaynegrahamdevicloud.onmicrosoft.com |
| Sync method | Microsoft Entra Cloud Sync (Provisioning Agent) |

---

## Implementation Steps

### 1. Provisioning Agent Installation

Downloaded and installed the Microsoft Entra Provisioning Agent on the Domain 
Controller. The setup wizard was used to connect the agent to both the 
on-premises Active Directory domain and the Entra ID tenant.

<img width="1241" height="875" alt="setupwizard" src="https://github.com/user-attachments/assets/27f37d36-15b3-4da1-b7c8-def7a6f5eef9" />

### 2. Service Account Configuration

A Group Managed Service Account (gMSA) was automatically created by the wizard 
using domain admin credentials. This account manages the synchronisation 
process between Active Directory and Entra ID.

<img width="1308" height="907" alt="gmsaaccount" src="https://github.com/user-attachments/assets/9b555f8b-c5c0-4a5f-a2a7-263a77fc4b4f" />

### 3. Agent Configuration Confirmation

The wizard confirmed the final configuration before committing — showing 
lab.local as the Active Directory source and the Entra ID tenant as the target.

<img width="1373" height="886" alt="confirmconfig" src="https://github.com/user-attachments/assets/af180871-3267-4228-a969-9b4063e21724" />

### 4. Cloud Sync Configuration in Entra

After the agent was installed, the sync configuration was completed in the 
Entra admin portal. The lab.local domain was selected, password hash sync was 
enabled, and the configuration was set to scope all users.

<img width="1641" height="595" alt="cloudsyncconfig" src="https://github.com/user-attachments/assets/23e1f11d-2724-4de2-86e9-f7478ac6f443" />

### 5. Network Configuration

Getting the provisioning agent to communicate with Microsoft's cloud endpoints 
required resolving a VirtualBox networking issue. The DC had two network 
adapters — one Host-only for internal communication with the client VM and one 
NAT for internet access. The adapters were in the wrong order causing the agent 
to attempt outbound connections through the Host-only adapter which has no 
internet access.

The fix involved swapping the adapter order so NAT was Adapter 1 and setting 
a static IP on the NAT adapter using VirtualBox's default NAT range 
(10.0.2.x / gateway 10.0.2.2). Interface metrics were also configured to 
ensure correct routing priority.


<img width="1372" height="868" alt="adaptersettings" src="https://github.com/user-attachments/assets/d17f9ff9-1e22-439b-abc2-ef90d7cbc26a" />

---

## Validation & Results

### On-Premises User in ADUC

KAYNE GRAHAM exists in the IT OU within the lab.local domain prior to sync.

<img width="1293" height="835" alt="aduclist" src="https://github.com/user-attachments/assets/222d8469-49ea-4fe7-a820-e59faa028335" />

### Successful Provisioning Logs

After the agent connected successfully, provisioning logs confirmed that users 
and groups from Active Directory were created and updated in Microsoft Entra ID.

<img width="1919" height="953" alt="provisioninglogs" src="https://github.com/user-attachments/assets/386c0f58-cdb8-4986-b3d6-a6041184462e" />

### Synced User in Entra ID

The Users list in Entra ID shows KAYNE GRAHAM with On-premises sync = Yes, 
confirming the identity was successfully synced from on-premises AD.

<img width="1656" height="811" alt="users" src="https://github.com/user-attachments/assets/18064507-af7d-4028-a97f-efc990995b40" />

### User Properties — On-Premises Sync Details

The user properties page confirms the full sync details including the 
distinguished name, SAM account name, on-premises domain and last sync time.

<img width="1656" height="879" alt="userproperties" src="https://github.com/user-attachments/assets/b149c071-6675-47d5-a6d7-046b84261803" />

---

## Troubleshooting

This lab involved significant troubleshooting before the sync was successful. 
Key issues encountered and resolved:

**HybridIdentityServiceNoActiveAgents error**
The provisioning agent service was set to Automatic (Delayed) startup which 
caused it to not be running when Entra attempted to communicate. Fixed by 
setting the service to Automatic and ensuring it was running before initiating 
sync.

**VirtualBox network adapter ordering**
The NAT adapter (internet) was configured as Adapter 2 and Host-only (internal) 
as Adapter 1. Windows prioritised Adapter 1 for outbound traffic meaning the 
agent could not reach Microsoft cloud endpoints. Fixed by swapping adapter 
order and configuring a static IP on the NAT adapter.

**APIPA address on NAT adapter**
After swapping adapters the NAT adapter was not receiving a DHCP lease from 
VirtualBox resulting in an APIPA address (169.254.x.x). Fixed by manually 
assigning a static IP using the VirtualBox NAT default range (10.0.2.15 / 
gateway 10.0.2.2).

**Performance counter error (Event ID 12009)**
The provisioning agent logged a performance counter initialisation error caused 
by a missing ntdsperf.dll. Attempted fixes included sfc /scannow, 
lodctr /R and DISM restore. Resolved after Windows updates and agent reinstall.

---

## Skills Developed

- Hybrid identity architecture
- Microsoft Entra Cloud Sync configuration
- Provisioning agent installation and troubleshooting
- VirtualBox network adapter configuration
- Windows Server networking and routing
- Identity synchronisation between on-premises AD and Entra ID
- Reading provisioning and event logs for troubleshooting

---

## Key Takeaways

Hybrid identity is one of the most common configurations in enterprise and MSP 
environments. Most organisations don't run purely on-premises or purely cloud — 
they run both. Understanding how the provisioning agent bridges the two and the 
networking requirements involved is directly applicable to real MSP work.

The troubleshooting process in this lab was as valuable as the configuration 
itself. Issues with service startup, network adapter priority and DHCP all 
required methodical diagnosis using ipconfig, ping, Test-NetConnection and 
Windows Event Viewer — tools used daily in IT support roles.

---

## Related Labs

- [microsoft-lab](https://github.com/kaynegraham/onprem-lab) — 
On-premises Active Directory
- [azure-lab](https://github.com/kaynegraham/azure-lab) — 
Microsoft Entra ID and Conditional Access

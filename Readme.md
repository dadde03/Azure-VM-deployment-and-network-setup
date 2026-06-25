# Azure NSG + SSH + Nginx Lab

## Overview
This lab documents a small but practical Azure Administrator project focused on understanding how Network Security Groups (NSGs) control inbound traffic to an Azure virtual machine and how that traffic maps to actual services running inside the guest operating system.[cite:81][cite:84] The environment uses a Linux VM, a virtual network and subnet, an NSG associated to the subnet, SSH access on port 22, and Nginx exposed on port 80 to validate network behavior from both a terminal and a browser.[cite:84][cite:188]

## Goal
The goal of the lab is to understand one AZ-104 concept deeply rather than touch many services superficially: inbound traffic control with Azure NSGs.[cite:84][cite:200] The project also reinforces the relationship between Azure network rules, public IP access, SSH authentication, and the guest OS service layer.[cite:85][cite:119]

## Learning objectives
By the end of the lab, the environment demonstrates how to:

- Create a virtual network and subnet for a VM deployment.[cite:81]
- Deploy a Linux VM with a public IP and access it remotely over SSH.[cite:44][cite:41]
- Associate an NSG to a subnet and control inbound traffic with custom rules.[cite:81][cite:84]
- Allow SSH on port 22 and HTTP on port 80 using explicit inbound rules.[cite:84][cite:196]
- Verify access from the client side and distinguish protocol behavior, such as why a browser cannot be used for SSH.[cite:44][cite:129]
- Troubleshoot connectivity issues by separating network reachability problems from authentication problems.[cite:85][cite:119]

## Architecture
The lab uses the following logical design:

- One resource group to contain all resources.[cite:69]
- One virtual network with one subnet for the VM.[cite:81]
- One Linux virtual machine with a public IP on the primary NIC.[cite:27][cite:48]
- One NSG associated with the subnet.[cite:81]
- One inbound rule for SSH on TCP 22.[cite:84]
- One inbound rule for HTTP on TCP 80.[cite:84][cite:196]
- One Nginx web server installed inside the VM to validate application traffic.[cite:188]

## Services used
| Service | Purpose |
|---------|---------|
| Azure Resource Group | Logical container for all lab resources.[cite:69] |
| Azure Virtual Network | Provides private address space for the VM.[cite:81] |
| Azure Subnet | Segments the network and receives the NSG association.[cite:81] |
| Azure Network Security Group | Filters inbound traffic based on rules, ports, and priority.[cite:84] |
| Azure Linux VM | Acts as the target compute resource for SSH and HTTP testing.[cite:27][cite:44] |
| Public IP | Enables direct inbound access from the internet to the VM.[cite:48][cite:44] |
| Nginx | Provides a simple HTTP service to validate port 80 connectivity.[cite:188] |

## Lab steps

### 1. Create the base network
A resource group was created first to hold all related resources in one place, followed by a virtual network and a subnet for the lab VM.[cite:69][cite:81] This established the network scope where NSG rules would later be applied.[cite:84]

### 2. Deploy the Linux VM
A Linux virtual machine was deployed into the existing subnet with a public IP attached to its primary NIC so it could be managed remotely.[cite:27][cite:44] SSH-based administration was chosen because it is the standard secure access method for Azure Linux VMs.[cite:41][cite:44]

### 3. Create and associate the NSG
An NSG was created and associated with the subnet instead of the NIC to centralize rule application at the subnet boundary.[cite:81][cite:84] This setup made it easier to understand how traffic is evaluated before it reaches the VM.[cite:200][cite:205]

### 4. Configure SSH access
An inbound rule was configured to allow TCP 22 so the VM could be reached over SSH.[cite:84][cite:85] During testing, a browser was incorrectly used against the public IP, which timed out because browsers speak HTTP or HTTPS rather than SSH.[cite:44][cite:129] Once the SSH client in PowerShell was used correctly, the connection reached the VM and exposed an authentication issue rather than a network issue.[cite:119]

### 5. Troubleshoot authentication
The SSH error changed from connection timeout to `Permission denied (publickey)`, which showed that port 22 was reachable and the remaining issue was authentication.[cite:119][cite:141] The connection succeeded only after using the correct private key and the correct SSH command syntax in PowerShell with the `-i` option.[cite:168][cite:169]

### 6. Install Nginx
After SSH access worked, the package index was updated with `sudo apt update`, and Nginx was installed with `sudo apt install nginx -y`.[cite:178][cite:188] This added a real application service inside the VM so the network test could move beyond administration traffic.[cite:188]

### 7. Open HTTP in the NSG
A second inbound rule was created to allow TCP 80 so web traffic could reach Nginx from the public internet.[cite:84][cite:196] This demonstrated that each workload port must be intentionally exposed through the NSG if external access is required.[cite:84]

### 8. Validate in the browser
Opening `http://<public-ip>` in a browser displayed the Nginx welcome page once port 80 was allowed and the service was running.[cite:188][cite:195] This confirmed the full chain: public IP, NSG rule, VM reachability, guest OS service, and application response.[cite:205]

## Validation performed
The lab was validated with two different client behaviors:

- SSH from PowerShell to verify administrative access on port 22.[cite:41][cite:168]
- Browser access to verify HTTP service delivery on port 80.[cite:188][cite:195]

A useful part of the validation process was seeing that the same public IP behaves differently depending on protocol and port, which is exactly how NSG-based traffic control should be understood.[cite:84][cite:200]

## Key concepts learned

### NSG rule evaluation
Azure processes NSG rules in priority order, and lower numbers have higher priority.[cite:81] This means a rule can exist and still not behave as expected if another rule with a higher priority number is evaluated first or if the effective rule set changes because of multiple associations.[cite:200][cite:205]

### Protocol awareness
A public IP does not imply that every client method is valid for every service.[cite:44] Browsers are used for HTTP and HTTPS, while SSH requires an SSH client and usually port 22.[cite:44][cite:129]

### Reachability vs authentication
A timeout usually indicates that traffic is not completing successfully to the destination service, while `Permission denied (publickey)` indicates that the connection reached the SSH service but authentication failed.[cite:119] This distinction is important because it tells the administrator whether to troubleshoot networking or credentials first.[cite:85][cite:119]

### Azure layer vs guest OS layer
Opening a port in Azure is necessary but not sufficient.[cite:84] The guest operating system must also be listening on the expected port, which is why Nginx had to be installed and running before HTTP validation could succeed.[cite:188][cite:195]

## Problems encountered
| Problem | Cause | Resolution |
|---------|-------|------------|
| Browser request to VM IP timed out | The browser was used against an SSH endpoint instead of an HTTP service.[cite:129] | Switched to an SSH client in PowerShell for port 22 access.[cite:41][cite:168] |
| `Permission denied (publickey)` during SSH | The connection reached the VM, but authentication did not match the configured key/user combination.[cite:119][cite:141] | Used the correct private key and SSH syntax with the `-i` option.[cite:168][cite:169] |
| HTTP not available initially | Port 80 was not yet exposed and Nginx was not yet part of the VM configuration.[cite:84][cite:188] | Installed Nginx and added an inbound NSG rule for TCP 80.[cite:188][cite:196] |

## Commands used
The following commands were used inside the process:

```bash
sudo apt update
sudo apt install nginx -y
systemctl status nginx
```

These commands update the package index, install the Nginx package, and verify whether the web server is active on the VM.[cite:178][cite:188]

PowerShell was used for SSH access with a private key using the `-i` option supported by Windows OpenSSH.[cite:168][cite:169]

## Suggested screenshots
To make the documentation complete, the following screenshots should be added:

- VM overview page showing name, status, OS, and public IP.
- NSG inbound rules showing SSH and HTTP rules.
- PowerShell terminal with successful SSH connection.
- Browser showing the Nginx welcome page.
- Optional effective security rules view for the VM.[cite:205]

## Result
The final environment successfully accepted SSH administration traffic on port 22 and served an Nginx web page on port 80 through the same public IP, with access controlled by NSG inbound rules.[cite:84][cite:188] This lab achieved the intended learning goal of deeply understanding how Azure network filtering interacts with real VM services and client protocols.[cite:200][cite:205]

## Lessons learned
- A public IP alone does not guarantee application access; the NSG and the guest OS service must both be configured correctly.[cite:84][cite:188]
- SSH and HTTP use different protocols and clients, even when they target the same VM IP.[cite:44][cite:129]
- Authentication errors and connectivity errors should be treated as different troubleshooting paths.[cite:85][cite:119]
- Small labs are effective for mastering AZ-104 concepts because they isolate one core area and make troubleshooting easier to understand.[cite:200][cite:205]

## Cleanup
After finishing the lab, the cleanest way to remove costs is to delete the entire resource group so that the VM, NSG, VNet, public IP, and related resources are removed together.[cite:69] This also matches the intended lifecycle design of Azure resource groups as containers for related resources.[cite:75]

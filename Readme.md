# Azure NSG + SSH + Nginx Lab

## Overview
This lab documents a small but practical Azure Administrator project focused on understanding how Network Security Groups (NSGs) control inbound traffic to an Azure virtual machine and how that traffic maps to actual services running inside the guest operating system. The environment uses a Linux VM, a virtual network and subnet, an NSG associated to the subnet, SSH access on port 22, and Nginx exposed on port 80 to validate network behavior from both a terminal and a browser.

## Goal
The goal of the lab is to understand one AZ-104 concept deeply rather than touch many services superficially: inbound traffic control with Azure NSGs. The project also reinforces the relationship between Azure network rules, public IP access, SSH authentication, and the guest OS service layer.

## Learning objectives
By the end of the lab, the environment demonstrates how to:

- Create a virtual network and subnet for a VM deployment.
- Deploy a Linux VM with a public IP and access it remotely over SSH.
- Associate an NSG to a subnet and control inbound traffic with custom rules.
- Allow SSH on port 22 and HTTP on port 80 using explicit inbound rules.
- Verify access from the client side and distinguish protocol behavior, such as why a browser cannot be used for SSH.
- Troubleshoot connectivity issues by separating network reachability problems from authentication problems.

## Architecture
The lab uses the following logical design:

- One resource group to contain all resources.
- One virtual network with one subnet for the VM.
- One Linux virtual machine with a public IP on the primary NIC.
- One NSG associated with the subnet.
- One inbound rule for SSH on TCP 22.
- One inbound rule for HTTP on TCP 80.
- One Nginx web server installed inside the VM to validate application traffic.

## Services used
| Service | Purpose |
|---------|---------|
| Azure Resource Group | Logical container for all lab resources.
| Azure Virtual Network | Provides private address space for the VM.
| Azure Subnet | Segments the network and receives the NSG association.
| Azure Network Security Group | Filters inbound traffic based on rules, ports, and priority.
| Azure Linux VM | Acts as the target compute resource for SSH and HTTP testing.
| Public IP | Enables direct inbound access from the internet to the VM.
| Nginx | Provides a simple HTTP service to validate port 80 connectivity.

## Lab steps

### 1. Create the base network
A resource group was created first to hold all related resources in one place, followed by a virtual network and a subnet for the lab VM. This established the network scope where NSG rules would later be applied.

### 2. Deploy the Linux VM
A Linux virtual machine was deployed into the existing subnet with a public IP attached to its primary NIC so it could be managed remotely. SSH-based administration was chosen because it is the standard secure access method for Azure Linux VMs.

### 3. Create and associate the NSG
An NSG was created and associated with the subnet instead of the NIC to centralize rule application at the subnet boundary.This setup made it easier to understand how traffic is evaluated before it reaches the VM.

### 4. Configure SSH access
An inbound rule was configured to allow TCP 22 so the VM could be reached over SSH. During testing, a browser was incorrectly used against the public IP, which timed out because browsers speak HTTP or HTTPS rather than SSH. Once the SSH client in PowerShell was used correctly, the connection reached the VM and exposed an authentication issue rather than a network issue.

### 5. Troubleshoot authentication
The SSH error changed from connection timeout to `Permission denied (publickey)`, which showed that port 22 was reachable and the remaining issue was authentication. The connection succeeded only after using the correct private key and the correct SSH command syntax in PowerShell with the `-i` option.

### 6. Install Nginx
After SSH access worked, the package index was updated with `sudo apt update`, and Nginx was installed with `sudo apt install nginx -y`. This added a real application service inside the VM so the network test could move beyond administration traffic.

### 7. Open HTTP in the NSG
A second inbound rule was created to allow TCP 80 so web traffic could reach Nginx from the public internet. This demonstrated that each workload port must be intentionally exposed through the NSG if external access is required.

### 8. Validate in the browser
Opening `http://<public-ip>` in a browser displayed the Nginx welcome page once port 80 was allowed and the service was running. This confirmed the full chain: public IP, NSG rule, VM reachability, guest OS service, and application response.

## Validation performed
The lab was validated with two different client behaviors:

- SSH from PowerShell to verify administrative access on port 22.
- Browser access to verify HTTP service delivery on port 80.

A useful part of the validation process was seeing that the same public IP behaves differently depending on protocol and port, which is exactly how NSG-based traffic control should be understood.

## Key concepts learned

### NSG rule evaluation
Azure processes NSG rules in priority order, and lower numbers have higher priority. This means a rule can exist and still not behave as expected if another rule with a higher priority number is evaluated first or if the effective rule set changes because of multiple associations.

### Protocol awareness
A public IP does not imply that every client method is valid for every service.Browsers are used for HTTP and HTTPS, while SSH requires an SSH client and usually port 22.

### Reachability vs authentication
A timeout usually indicates that traffic is not completing successfully to the destination service, while `Permission denied (publickey)` indicates that the connection reached the SSH service but authentication failed. This distinction is important because it tells the administrator whether to troubleshoot networking or credentials first.

### Azure layer vs guest OS layer
Opening a port in Azure is necessary but not sufficient. The guest operating system must also be listening on the expected port, which is why Nginx had to be installed and running before HTTP validation could succeed.

## Problems encountered
| Problem | Cause | Resolution |
|---------|-------|------------|
| Browser request to VM IP timed out | The browser was used against an SSH endpoint instead of an HTTP service.| Switched to an SSH client in PowerShell for port 22 access. |
| `Permission denied (publickey)` during SSH | The connection reached the VM, but authentication did not match the configured key/user combination. | Used the correct private key and SSH syntax with the `-i` option. |
| HTTP not available initially | Port 80 was not yet exposed and Nginx was not yet part of the VM configuration. | Installed Nginx and added an inbound NSG rule for TCP 80. |

## Commands used
The following commands were used inside the process:

```bash
sudo apt update
sudo apt install nginx -y
systemctl status nginx
```

These commands update the package index, install the Nginx package, and verify whether the web server is active on the VM.

PowerShell was used for SSH access with a private key using the `-i` option supported by Windows OpenSSH.

## Suggested screenshots
To make the documentation complete, the following screenshots should be added:

- VM overview page showing name, status, OS, and public IP.
- NSG inbound rules showing SSH and HTTP rules.
- PowerShell terminal with successful SSH connection.
- Browser showing the Nginx welcome page.
- Optional effective security rules view for the VM.

## Result
The final environment successfully accepted SSH administration traffic on port 22 and served an Nginx web page on port 80 through the same public IP, with access controlled by NSG inbound rules. This lab achieved the intended learning goal of deeply understanding how Azure network filtering interacts with real VM services and client protocols.

## Lessons learned
- A public IP alone does not guarantee application access; the NSG and the guest OS service must both be configured correctly.
- SSH and HTTP use different protocols and clients, even when they target the same VM IP.
- Authentication errors and connectivity errors should be treated as different troubleshooting paths.
- Small labs are effective for mastering AZ-104 concepts because they isolate one core area and make troubleshooting easier to understand.

## Cleanup
After finishing the lab, the cleanest way to remove costs is to delete the entire resource group so that the VM, NSG, VNet, public IP, and related resources are removed together. This also matches the intended lifecycle design of Azure resource groups as containers for related resources.

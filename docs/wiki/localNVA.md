# Creating a Local NVA in Azure for Oracle Database@Azure

## Benefits

This setup helps solve scenarios where traffic inspection is required between Oracle Database on Azure and other resources. It is also useful for specific scenarios where native support is not available, such as with private endpoints or some services with delegated subnets.

## Note

Although we use the term "local NVA," this node is a Linux VM with IP forwarding enabled, and not intended to be an enterprise-scale Firewall NVA.

## 1. Create a Linux VM in Azure as an NVA

- **Set up a Linux VM** (can be any of the supported distributions on Azure) in the desired resource group and region using the Azure portal or CLI.
- **Ensure the VM is in the same Virtual Network, but separate subnet, as the Oracle Database**.

### Note

Sizing is very much driven by the actual traffic pattern. Consider how much traffic (volume), packets per second, etc., are involved. Starting with a 2-core general-purpose VM (such as a `D2s_v5` with 2 vCPUs and 8 GiB memory) may be used to gauge initial performance. High storage/IOPS performance SKUs are not necessary for this use case.

## 2. Enable IP Forwarding on the VM's NIC

- Go to the **Networking** section of the NVA VM.
- Select the **Network Interface**.
- Under **Settings**, choose **IP configurations**.
- Enable **IP forwarding**.

## 3. Enable IP Forwarding at the OS Level

- SSH into the VM.
- Edit the sysctl configuration to enable IP forwarding:
  - `sudo nano /etc/sysctl.conf`
  - Add or uncomment the line:
    - `net.ipv4.ip_forward = 1`
- Apply the changes
- Run the following command to reset the network status to forward network traffic without a reboot:
  - `sudo sysctl -p`
- Ensure that the local firewall on the NVA is not enabled or set to block traffic.

## 4. Configure Route Tables

- **Create a route table** in the Azure portal.
- **Add routes** to the route table:
  - **To Oracle Database Subnet**: Set the next hop to the local NVA VM.
  - **From Oracle Database Subnet**: Set the next hop to the local NVA VM.
- **Associate the route table** with the appropriate subnets.

## Summary of Steps

1. **Create Linux VM** in Azure in the same VNet as the Oracle Database.
2. **Enable IP forwarding** on the VM's NIC.
3. **Enable IP forwarding** on the Linux VM at the OS level.
4. **Configure route tables** to use the NVA as the first hop for traffic to and from the Oracle Database subnet.

This setup ensures that all traffic to and from the Oracle Database goes through your local NVA.

## Example Lab: Hub-Spoke Environment

### Environment Overview

- **Hub VNet (10.0.0.0/16)**

  - Hub NVA: 10.0.0.4

- **Spoke 1 VNet - Application Tier (10.1.0.0/16)**

  - Application Server: 10.1.0.4

- **Spoke 2 VNet - Oracle DB (10.2.0.0/16)**
  - Oracle DB Subnet: 10.2.0.0/24
  - Oracle Database: 10.2.0.4
  - Local NVA Subnet: 10.2.1.0/24
  - Local NVA: 10.2.1.4

### Route Configuration

#### Application Tier VNet (Spoke 1)

1. **Route to Hub NVA:**
   - Destination: 10.2.0.0/24 (Oracle DB Subnet)
   - Next Hop: 10.0.0.4 (Hub NVA)
   - Attach to Client/Application Subnet

#### Hub VNet

1. **Route to Local NVA:**
   - Destination: 10.2.0.0/24 (Oracle Subnet)
   - Next Hop: 10.2.1.4 (Local NVA)
   - Attached to Hub NVA Subnet

#### Oracle DB VNet (Spoke 2)

1. **Route to Local NVA:**

   - Destination: 10.1.0.0/16 (Application Tier VNet)
   - Next Hop: 10.2.1.4 (Local NVA)
   - Attach to Oracle DB Subnet

2. **Route from Local NVA to Application Tier VNet:**
   - Destination: 10.1.0.0/16 (Application Tier VNet)
   - Next Hop: 10.0.0.4 (Hub NVA)
   - Attach to Local NVA Subnet

# Creating a Local NVA in Azure for Oracle Database@Azure

## Benefits

This setup helps solve scenarios where traffic inspection is required between Oracle Database on Azure and other resources. It is also useful for specific scenarios where native support is not available, such as with private endpoints or some services with delegated subnets.

> **Note**  
> Although we use the term "local NVA," this node is a Linux VM with IP forwarding enabled, and not intended to be an enterprise-scale Firewall NVA.

## 1. Create a Linux VM in Azure as an NVA

- **Set up a Linux VM** in the desired resource group and region using the Azure portal or CLI. This guide uses Oracle Linux (version 8.8), but you can use any supported Linux distribution on Azure. Be aware that some commands or paths might differ depending on the distribution you choose.
- **Ensure the VM is in the same Virtual Network, but separate subnet, as the Oracle Database**.

> **Note**  
> Sizing is very much driven by the actual traffic pattern. Consider how much traffic (volume), packets per second, etc., are involved. Starting with a 2-core general-purpose VM (such as a `D2s_v5` with 2 vCPUs and 8 GiB memory) may be used to gauge initial performance. High storage/IOPS performance SKUs are not necessary for this use case.

## 2. Enable IP Forwarding on the VM's NIC

- Go to the **Networking** section of the NVA VM.
- Select the **Network Interface**.
- Enable **IP forwarding**.

## 3. Enable IP Forwarding at the OS Level

- SSH into the VM.
- Edit the sysctl configuration to enable IP forwarding:
  ```bash
  sudo nano /etc/sysctl.conf
  ```
  - Add or uncomment the line:
    - `net.ipv4.ip_forward = 1`
- Apply the changes
- Run the following command to reset the network status to forward network traffic without a reboot:

  ```bash
  sudo sysctl -p
  ```

- Disable the Firewall on the Local NVA if enabled:
  ```bash
  sudo systemctl status firewalld
  sudo systemctl stop firewalld
  sudo systemctl disable firewalld
  ```

## 4. Configure Route Tables

You need to create and configure route tables for each VNet involved, with different configurations depending on whether VNets are directly peered or connected via a hub/spoke model.

- **Create or modify a route table** for each VNet involved:

  - **For directly peered VNets**:

    - **In the Application Tier VNet**:
      - Add a route for traffic destined to the Oracle DB subnet, setting the next hop to the local NVA VM in the Oracle DB VNet.
    - **In the Oracle DB VNet**:
      - Add a route for traffic destined to the Application Tier VNet, setting the next hop to the local NVA VM in the Oracle DB VNet.

  - **For VNets connected via a hub/spoke model**:
    - **In the Hub VNet**:
      - Add routes for traffic destined to the Oracle DB subnet, setting the next hop to the local NVA VM in the Oracle DB VNet.
    - **In the Application Tier VNet**:
      - Add a route for traffic destined to the Oracle DB subnet, setting the next hop to the Hub NVA in the Hub VNet. (This may already be in place.)
    - **In the Oracle DB VNet**:
      - On the Oracle DB subnet, add a route for traffic destined to the Application Tier VNet, setting the next hop to the local NVA VM.
      - On the local NVA subnet, add a route for traffic destined to the Application Tier VNet, setting the next hop to the Hub NVA in the Hub VNet.

## Summary of Steps

1. **Create Linux VM** in Azure in the same VNet as the Oracle Database.
2. **Enable IP forwarding** on the VM's NIC.
3. **Enable IP forwarding** on the Linux VM at the OS level.
4. **Disable the firewall** on the local NVA if enabled
5. **Configure route tables** to use the NVA as the first hop for traffic to and from the Oracle Database subnet.

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

1. **Route to Hub NVA**:
   - Destination: 0.0.0.0/0 (All traffic)
   - Next Hop: 10.0.0.4 (Hub NVA)
   - Attach to Client/Application Subnet

#### Hub VNet

1. **Route to Oracle DB via Local NVA**:
   - Destination: 10.2.0.0/24 (Oracle DB Subnet)
   - Next Hop: 10.2.1.4 (Local NVA)
   - Attach to Hub NVA Subnet

#### Oracle DB VNet (Spoke 2)

1. **Route to Application Tier via Local NVA**:

   - Destination: 10.1.0.0/16 (Application Tier VNet)
   - Next Hop: 10.2.1.4 (Local NVA)
   - Attach to Oracle DB Subnet

2. **Route from Local NVA to Application Tier via Hub NVA**:
   - Destination: 0.0.0.0/0 (All traffic)
   - Next Hop: 10.0.0.4 (Hub NVA)
   - Attach to Local NVA Subnet

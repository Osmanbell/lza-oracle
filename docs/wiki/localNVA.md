# Creating a Local NVA in Azure for Oracle Database@Azure

## Benefits

This setup helps solve scenarios where traffic inspection is required between Oracle Database on Azure and other resources. It is also useful for specific scenarios where native support is not available, such as with private endpoints or some services with delegated subnets.

## Note

Although we use the term "local NVA," this node is a Linux VM with IP forwarding enabled, and not intended to be an enterprise-scale Firewall NVA.

## 1. Create a Linux VM in Azure as an NVA

- **Set up a Linux VM** (can be any of the supported distributions on Azure) in the desired resource group and region using the Azure portal or CLI.
- **Ensure the VM is in the same Virtual Network, but separate subnet, as the Oracle Database**.

## 2. Enable IP Forwarding on the VM's NIC

- Go to the **Networking** section of the NVA VM.
- Select the **Network Interface**.
- Under **Settings**, choose **IP configurations**.
- Enable **IP forwarding**.

## 3. Enable IP Forwarding at the OS Level

- SSH into the VM.
- Run the following commands to enable IP forwarding:
  - sudo sysctl -w net.ipv4.ip_forward=1
  - sudo sysctl --system
- Apply the changes.
- Run the following command to reset the network status to forward network traffic without a reboot:
  - systemctl -p
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

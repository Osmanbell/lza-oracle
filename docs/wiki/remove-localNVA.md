# Removing the Local NVA Workaround and Configuring the Hub NVA for Oracle Database Traffic Inspection

## Benefits

For the East US region, the local NVA workaround is no longer necessary for scenarios where traffic inspection is required to/from Oracle Database and using a third-party firewall (not Azure Firewall). This updated setup will streamline network management and improve efficiency by centralizing traffic inspection through the Hub NVA.

## Note

This does not remove the need for a local NVA for scenarios involving private endpoints or serverless applications, such as Azure Web Apps and Azure Functions.

## Steps to Remove Local NVA

### 1. Important Considerations

- **No Public IP for Hub NVA**:

  - Ensure that the Hub NVA does not have a public IP assigned to it.

- **Subscription Onboarding for New Network Stack**:
  - Work with Azure Product Group for support on onboarding the subscription to the updated networking stack.

### 2. Update Route Tables to Use the Hub NVA

- **Modify Route Table**:
  - Remove any routes previously associated with the local NVA.
  - **To Oracle Database Subnet**: Set the next hop to the Hub NVA.
  - **From Oracle Database Subnet**: Set the next hop to the Hub NVA.

### 3. Configure the Hub NVA

- **Stop and Deallocate the Hub NVA**:

  - Go to the Azure portal.
  - Navigate to the Hub NVA VM.
  - Click on **Stop** to stop and deallocate the VM.

- **Start the Hub NVA**:
  - Once the VM is stopped and deallocated, click on **Start** to start the VM again.

### 4. Decommission the Local NVA

- **Delete the Local NVA VM**:

  - Navigate to the Azure portal.
  - Find the local NVA VM in the resource group.
  - Click on **Delete** to remove the VM.

- **Remove Associated Resources**:
  - Delete any associated resources, such as disks, NICs, and public IPs, if any.

## Summary of Steps

1. **Ensure all considerations** are accounted for.
2. **Onboard the subscription** to the new network stack via Azure Product Group.
3. **Update route tables** to use the Hub NVA as the next hop for traffic to and from the Oracle Database subnet. Remove any routes associated with the Local NVA.
4. **Stop, deallocate, and start the Hub NVA** to ensure it is onboarded to the updated networking stack.
5. **Decommission the local NVA** by deleting the VM and associated resources.

This process will enable traffic inspection on a third-party Hub NVA to/from Oracle Database@Azure natively, without the use of a local NVA workaround, simplifying management and reducing overhead.

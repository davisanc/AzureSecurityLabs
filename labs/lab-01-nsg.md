# Lab 1 - Protecting the Network Perimeter

## Utilising NSG (Network Security Groups)

Network Security Groups filter traffic to and from resources in an Azure virtual network. They can be applied at subnet or virtual machine level, and filter the traffic based on a set of rules.

Further information on network security groups (NSG) can be found in the Azure documentation:

[https://docs.microsoft.com/en-us/azure/virtual-network/security-overview](https://docs.microsoft.com/en-us/azure/virtual-network/security-overview)

You can use security groups to protect/restrict traffic between tiers. In the 3-tier architecture shown, the web tier should not communicate directly with any resource in the database tier. To enforce this, the database subnet should block all incoming traffic from the web tier subnet. This can be done using a security group.

### 1.1 - Creating the security group
This section creates the security group to protect the database tier.

1. In the VS Code terminal, enter the following CLI command to create the security group **SQL-NSG**
    ```
    az network nsg create --name SQL-NSG --resource-group <resource-group-name> --location <location>
    ```

    When the command completes, the terminal will show all properties of the security group, and you can also see the new NSG in the Azure console.

    By default, a security group will be pre-populated with three **inbound** rules (in order of execution):
    1. allow any VNet to VNet traffic
    2. allow any traffic from the Azure load balancer to the VNet
    3. deny all inbound traffic (that does not match any other rule)

    There are also three **outbound** rules:
    1. allow any traffic outbound to the VNet
    2. allow all traffic outbound to the Internet
    3. deny all outbound traffic

    You can view these rules in the Azure Console by selecting the security group object within the Resource Group, and also by running this CLI command:
    ```
    az network nsg show --resource-group <resource-group-name> --name SQL-NSG --query "defaultSecurityRules[]" --output table
    ```

    These rules cannot be deleted. What we can do is create a new series of rules in the security group with a higher priority to filter the traffic before the default rules are evaluated.

2.  Allow inbound traffic from the business tier subnet.

    Create a new rule within your new security group. Like typical firewall rules, security group rules are based on information about the source, destination, protocol, port and action (allow/deny).

    This rule will allow any traffic into the database tier from the business tier. This is done by specifying the CIDR of the business tier subnet (10.0.2.0/24) as the source. Any resource within this subnet can communicate with the database tier.

    ```
    az network nsg rule create --name AllowFromBiz --nsg-name SQL-NSG --priority 110 --resource-group <resource-group-name> --description "Allow all traffic from the Business Tier" --access Allow --direction Inbound --source-address-prefix 10.0.2.0/24 --source-port-ranges * --protocol * --destination-address-prefix * --destination-port-ranges *
    ```

3. Allow inbound traffic from the database tier subnet itself.

    This rule allows communication between the database VMs, which is needed for database replication and failover. Use the database (SQL) subnet range 10.0.3.0/24 as the source:

    ```
    az network nsg rule create --name AllowFromSQL --nsg-name SQL-NSG --priority 120 --resource-group <resource-group-name> --description "Allow all intra SQL traffic within the Database Tier" --access Allow --direction Inbound --source-address-prefix 10.0.3.0/24 --source-port-ranges * --protocol * --destination-address-prefix * --destination-port-ranges *
    ```

4. Allow RDP access from the Jump Box.

    Allowing RDP traffic on the RDP port (3389) from the Jump Box lets remote administrators connect to the database servers from the Jump Box. By only specifying the RDP port, users of the Jump Box will not be able to connect via any other method, port or protocol.

    Use the management subnet range 10.0.0.128/25 as the source:

    ```
    az network nsg rule create --name AllowRDPFromJB --nsg-name SQL-NSG --priority 130 --resource-group <resource-group-name> --description "Allow RDP traffic from the Jump Box" --access Allow --direction Inbound --source-address-prefix 10.0.0.128/25 --source-port-ranges * --protocol TCP --destination-address-prefix * --destination-port-ranges 3389
    ```

5. Deny all other inbound traffic from the VNet.

    Now we have set up the base access requirements, we need to block all other traffic from within the VNet. Instead of a source address, we can use the **VirtualNetwork** tag in the rule:

    ```
    az network nsg rule create --name DenyFromVNet --nsg-name SQL-NSG --priority 140 --resource-group <resource-group-name> --description "Deny general VNet traffic" --access Deny --direction Inbound --source-address-prefix VirtualNetwork --destination-port-ranges *
    ```

6. Deny all inbound traffic from the Internet.

    Create the final rule yourself using the commands above as the pattern. Set your own description, and use these properties:

    - **Name:** DenyFromInternet
    - **Priority:** 150
    - **Source Address:** Internet

#### Rule Priority
Consider the following when creating security group rules...

- Security group rules run in priority order, with the lowest priority rule being evaluated first.
- Leave a reasonable gap between your rules. It makes for a lot of work to try and retro-fit a new rule with a higher priority in between rules priorties 4,5 and 6 than 140, 150 and 160.
- The first **Deny** rule encountered by the evaluation instantly deny the access.

### 1.2 - Attach the security group to the SQL Server network interface / NIC

Follow these steps to attach the new NSG to the network interface of the SQL VM...

![NSG-inbound-sql](/images/attach-NSG-NIC.PNG)

Alternatively you can use the following CLI command to make this attachment. The command references the NIC resource and attaches the security group to the NIC.

**Note:** The name of the NIC for the SQL server is **sql-vm1-nic1**

```
az network nic update --resource-group <resource-group-name> --name sql-vm1-nic1 --network-security-group SQL-NSG
```

### 1.3 - Overview and final steps

Your NSG rule set should look similar to this...

![NSG-inbound-sql](/images/NSG-SQL-NIC.PNG)

Confirm that you can RDP from the Jump Box to the SQL server and also from the Business VM but not from the Web VM (you will need to RDP to the Jump Box first, then RDP to the Business and Web VMs to finally RDP to the SQL VM)

![RDP blocked](/images/RDP-blocked-from-web.PNG)

<< [Back to home page](/README.md)

<< [Back to Lab Index](README.md) . . . . . [Next Lab](lab-02-networking-logs.md) >>

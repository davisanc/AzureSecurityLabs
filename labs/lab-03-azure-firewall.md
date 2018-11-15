# Lab 3 - Control outbound security traffic with Azure Firewall

## Lab Overview

Azure Firewall is a stateful firewall as a service, with high availability and cloud scalability built-in. The primary use case for the Azure Firewall is to centrally create, enforce and log application and network policies. As the **first version** of the product, the firewall is focused on **securing outbound flows by FQDN whitelisting/blacklisting**. It provides source network address translation and it is integrated with Azure Monitor for logging and analytics.

### 3.1 - Enabling Azure Firewall preview

The Azure Firewall is currently in public preview. To enable the firewall in your subscription you need use the following Azure PowerShell commands:

To enable the firewall in your subscription you need use the following Azure PowerShell commands:

1. Run this command from PS

```PowerShell
Register-AzureRmProviderFeature -FeatureName AllowRegionalGatewayManagerForSecureGateway -ProviderNamespace Microsoft.Network
```

*Note: If you get the following error:*

```PowerShell
Powershell error – Import-Module : File AzureRM.psm1 cannot be loaded because running scripts is disabled on this system
```

*Run this command:*

```PowerShell
Set-ExecutionPolicy RemoteSigned
```

You may need to login again to your Azure subscription with **Connect-AzureRmAccount** or **az login**. Make sure you are on the right subscription.

2. Now run the following PS command needed to enable the Firewall

```PowerShell
Register-AzureRmProviderFeature -FeatureName AllowAzureFirewall -ProviderNamespace Microsoft.Network
```

**It takes up to 30 minutes for feature registration to complete**. You can check your registration status by running the following Azure PowerShell commands:

```PowerShell
Get-AzureRmProviderFeature -FeatureName AllowRegionalGatewayManagerForSecureGateway -ProviderNamespace Microsoft.Network
```

```PowerShell
Get-AzureRmProviderFeature -FeatureName AllowAzureFirewall -ProviderNamespace Microsoft.Network
```

After the registration is complete, run the following PowerShell command:

```PowerShell
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network
```

### 3.2 - Create a subnet for the firewall

The cloud firewall needs to have a dedicated subnet within your VNet named **AzureFirewallSubnet**. 

On the portal create a new subnet in the **ra-ntier-vnet** VNet named **AzureFirewallSubnet** with the IP address space of **10.0.6.0/24**.

Alternatively, run the following CLI command to create the subnet:

```PowerShell
az network vnet subnet create --address-prefix 10.0.3.0/24 --name AzureFirewallSubnet --resource-group <resource-group-name> --vnet-name ra-ntier-vnet
```

### 3.3 - Create the firewall

On the Azure Portal, click **Create a resource**, **Networking**, and look for **Firewall**. Configure the firewall as per the image below...

![firewall](/images/firewall.PNG)

Once the firewall object is created, view the properties of the firewall and take a note of the private IP address.

### 3.4 Routing the web-tier traffic
We will work on the Web VM, and we will set the default route of the web-tier subnet to send all traffic through the firewall.

1. Create a new Route Table

    ![route table](/images/route-table.PNG)

    The Azure CLI command to create the route table is:

    ```bash
    az network route-table create --name Firewall-Route --resource-group <resource-group-name> --location <location>
    ```

2. Create a default route

    To create the default route in the Azure Portal follow these steps. (The Azure CLI command follows these steps if you prefer.)

    1. In the Azure portal, locate and click on the route table resource created in the step above.
    2. Click **Routes** in the options on the left.
    3. Click **Add**.
    4. Create the route with the following properties:
        - **Route name:** FW-DG
        - **Address prefix:** 0.0.0.0/0
        - **Next Hop Type:** Select **Virtual Appliance**. Azure Firewall is actually a managed service, but **Virtual Appliance** works in this situation.
        - **Next Hop Address:** type the private IP address for the firewall created above.
        - Click **OK**.

    The Azure CLI command to create the default route is:

    ```Bash
    az network route-table route create --resource-group <resource-group-name> --route-table-name Firewall-Route --name FW-DG --address-prefix 0.0.0.0/0 --next-hop-type VirtualAppliance --next-hop-ip-address <firewall-ip-address>
    ```

3. Attach the route table to the web subnet

    1. Click **Subnets** in the list options for the route table.
    2. Click **Associate**.
    3. Click **Virtual network** and select **ra-ntier-vnet** from the list of VNets.
    4. Choose the **web** subnet from the list given, and click **OK**.

    To achieve this using the Azure CLI:

    ```Bash
    az network vnet subnet update --name web --resource-group <resource-group-name> --vnet-name ra-ntier-vnet --route-table Firewall-Route
    ```

    You may notice that the CLI method attaches the route table to the subnet inside the VNet using the 'network vnet' command subset, rather than setting this as a property on the route table object itself.

### 3.5 - Create application rules

We will write a simple rule to enable web traffic to *github.com* and block anything else. The rule uses the CIDR of the web subnet as the source address, allowing any resources within that tier access to *github.com*. Therefore existing *and* future VMs will be allowed to communicate via this rule.

1. Click on the firewall resource.
2. Under settings, click **Rules**.
3. Click **Application rule collection**  and the click **+ Add application rule collection**.
4. Use the following settings for the new collection...
    - **Name:** App-Coll01
    - **Priority:** 200
    - **Action:** Allow
    - Add the first rule...
        - **Name:** AllowGH
        - **Source Addresses:** 10.0.1.0/24
        - **Protocol:Port:** http,https
        - **Target FQDNs:** github.com
    - Click **Add**

After a short time the new application rule will appear in the firewall.

*Note:Azure Firewall includes a built-in rule collection for infrastructure FQDNs that are allowed by default. These FQDNs are specific for the platform and cannot be used for other purposes. The allowed infrastructure FQDNs include:*

- *Compute access to storage Platform Image Repository (PIR).*
- *Managed disks status storage access.*
- *Windows Diagnostics.*

*You can override this build-in infrastructure rule collection by creating a 'deny all' application rule collection which is processed last. It will always be processed before the infrastructure rule collection. Anything not in the infrastructure rule collection is denied by default.*

### 3.6 - Configure network rules

The idea is to permit DNS traffic to our DNS server to go through the Firewall (from a Level3/Level4 perspective).

1. On the firewall resource, under **Rules**, click **Network rule collection**.
2. Click **+ Add network rule collection**.
3. Use these setting for the new collection...
    - **Name:** Net-Coll01
    - **Priority** 200
    - **Action:** Allow
    - Add the first rule...
        - **Name:** AllowDNS
        - **Protocol:** select UDP
        - **Source Addresses:** type 10.0.1.0/24
        - **Destination address:** type 168.63.129.16, the IP address for internal DNS resolution
        - **Destination Ports:** 53.
    - Click **Add**.

### 3.7 - Test your firewall rules

RDP to the JumpBox and from there to the Web VM. Open a browser and try go to github.com.

![firewall allowed](/images/Firewall-allowed-rule.PNG)

Try going to another site, this action should be blocked:

![firewall blocked](/images/Fireall-blocked-rule.PNG)

![bbc blocked](/images/bbc.PNG)

<< [Back to home page](/README.md)

<< [Previous Lab](lab-02-networking-logs.md) . . . . . [Next Lab](lab-04-app-gateway.md) >>
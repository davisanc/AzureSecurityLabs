# AzureSecurityLabs

These series of labs are based on the Azure Reference Architecture to deploy an N-Tier application
https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/n-tier/n-tier-sql-server

![image of 3tier](/images/n-tier-sql-server.png)

## Lab Series

1.	Redeem your Azure Passes and activate your credit
2.	Configuration needed before starting the labs
3.	Deploy the solution
4.	Lab 1-  Protecting the Network Perimeter – NSG (Network security groups)
5.	Lab 2 - Azure Networking logs
6.	Lab 3 – Control outbound security traffic with Azure Firewall
7.	Lab 4 – Protecting the Web Application - Application Gateway and WAF (Web Application Firewall)
8.	Lab 5 – Understand your application security posture in Azure -Azure Security Center for security recommendations
9.	Lab 6 – Storage Security – Encryption at Rest - Apply disk encryption to a running VM
10.	Lab 7 – Extending your Data Centre to Azure in a secure way – Site to Site VPN Access
11.	Lab 8: Azure Active Directory Role-Based Access Control
12.	Lab 9: Enable DDoS protection for your resources

### 1.	Redeem your Azure Passes if you will use your Personal Microsoft Account

Note: We recommend using an enterprise subscription with a working account that have privileges to create resources. There are some labs on this guide that require the use or 3rd party Virtual Machines that don’t allow the use of Azure pass credits. In order to minimise costs we will enable ‘auto’shutdown’ of virtual machines. 

Make sure you have a working Microsoft account that you can use to redeem the Azure pass

Open a browser in private/incognito mode and go to https://microsoftazurepass.com

![image or redeem](/images/redeem-your-azure-pass.PNG)

Login with your personal account, confirm your details and click next

Enter the promo code of your Azure Pass

Activate your pass

![active pass](/images/activate-azure-pass.PNG)

It will take a few minutes until Azure sets up your new subscription

![setting subs](/images/azure-pass-setting-subscription.PNG)

Next when you log into https://portal.azure.com , go to Cost Management + Billing and you will be able to see your Azure pass credit

### 2.  Configuration needed before starting the labs (Time to complete: 15 min)

1.	Visual Studio Code 
- Install vscode from https://code.visualstudio.com
2.	PowerShell (we need PS version 6)
- Install the Azure PowerShell module
- Make sure you have installed PS version 6 or higher
```
Get-Module AzureRM -ListAvailable | Select-Object -Property Name,Version,Path
```
3. Install Azure CLI 2.0
- Open a Command Prompt and check that az produces command help output (try closing Windows Powershell and Visual Studio Code and re-open again)
- Note: in case you face an issue when you try to run an az command it says "az : The term 'az' is not recognized as the name of a cmdlet, function, script file, or operable program." The issue is because the azure cli 2.0 is instled in location - C:\Users\<username>\AppData\Local\Programs\Python\Python37-32\Scripts\  and this path isn't added to the PATH variable. 
First make sure you have python installed in your machine. If you don’t have the original CLI (or python) at all, you need that first. Download and install it from here: https://www.python.org/downloads/release/python-352/
1.	Uninstall Azure CLI earlier versions with command - pip uninstall azure-cli
2.	Re-install Azure CLI 2.0 - pip install --user azure-cli
3.	Add the path C:\Users\<username>\AppData\Local\Programs\Python\Python37-32\Scripts\ to PATH 
- check if the az command is working: az --help
4.	(Optional) On Visual Studio, go to Extensions, search for Azure CLI Tools and install the package
5.	Install the Azure building blocks npm package.
- Install node.js
- (you may need to close and re-open again PowerShell and VSC) 
  ```
  npm install -g @mspnp/azure-building-blocks
  ```
- Test Azure Building Blocks with the following command on PowerShell or VSC:
  ```
  azbb
  ```
6.	From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:
  ```
  Az login
  ```
7.	Make sure you use the right subscription (enterprise or Azure pass)
  ```
  az account set --subscription  "<subscription-ID>"
  ```

### 3.  Deploy the Solution (Time to complete: 50 min)

1.	Run the following command to create a resource group (use ‘uksouth’ for location)
  ```
  az group create --location <location> --name <resource-group-name>
  ```
2.	Run the following command to create a Storage account for your Cloud resources.
  ```
  az storage account create --location <location> \ --name <storage-account-name> \ --resource-group <resource-group-name> \ --sku Standard_LRS
  ```
3.	Navigate to https://github.com/davisanc/AzureSecurityLabs 

4.	Open the n-tier-windows-security-labs.json file

5.	In the json file, search for all instances with passwords. Replace them with your own admin users and passwords and save the file

6.	Run azbb command to set up through an ARM template the basics of the lab: jumpbox web VM, app VM and SQL VM:
    ```
    azbb -s <Subscription-ID> -g <Resource-Group-Name> -l <Location> -p .\<name-of-your-tempalte.json> --deploy
    ```
Note: this environment will need about 50 minutes to deploy. Once you have run the last commands report to your proctors you have got to this point

The Application and Database tier have an internal load balancer in front of them, so you can scale up the tier with more VMs if needed and the LB will distribute the traffic accordingly

The Web Tier doesn’t have any LB, as we will later create an External Application Gateway in front of the web tier

For now, the only Internet access to the environment is through the JumpBox as it’s the only VM with a public IP address

Test: make sure you can ping from the JB to the Web, Biz and DB virtual machines (enable PING on the firewall settings)

### 4.  Lab 1-  Protecting the Network Perimeter – NSG (Network security groups)

Create an NSG rule to restrict traffic between tiers. For example, in the 3-tier architecture shown, the web tier does not communicate directly with the database tier. To enforce this, the database tier should block incoming traffic from the web tier subnet

1.	Deny all inbound traffic from the VNet. (Use the VIRTUAL_NETWORK tag in the rule.)
2.	Allow inbound traffic from the business tier subnet.
3.	Allow inbound traffic from the database tier subnet itself. This rule allows communication between the database VMs, which is needed for database replication and failover.
4.	Allow RDP traffic (port 3389) from the jumpbox subnet. This rule lets administrators connect to the database tier from the jumpbox.
5.	Deny all inbound traffic from Internet. Use the “Internet” tag in the rule

Create rules 2 – 4 with higher priority than the first rule, so they override it

Apply your NSG to the JumpBox VM NIC

![NSG-inbound-sql](/images/NSG-inbound-rules-for-SQL.PNG)

Confirm that you can RDP from the Jumpbox to the SQL server and also from the Business VM (by the specific NSG added)  but not from the Web VM

![RDP blocked](/images/RDP-blocked-from-web.PNG)

### 5. Azure Networking logs

Network logging and monitoring in Azure is comprehensive and covers two broad categories:
- Network Watcher: Scenario-based network monitoring is provided with the features in Network Watcher. This service includes packet capture, next hop, IP flow verify, security group view, NSG flow logs. Scenario level monitoring provides an end to end view of network resources in contrast to individual network resource monitoring.
- Resource monitoring: Resource level monitoring comprises four features, diagnostics logs, metrics, troubleshooting, and resource health. All these features are built at the network resource level.

To troubleshoot your NSG rules, enable NSG flow logs. This will enable Network watcher. You will need to select an storage account within your Resource Group. The NSG flow logs will be stored within a blob container of the selected storage account. More details https://docs.microsoft.com/en-us/azure/network-watcher/network-watcher-nsg-flow-logging-portal

![NSG flow logs](/images/NSG-flow-logs-enabled.PNG)

Also, you may want to enable Traffic Analytics for rich analytics and visualization. You will need to create an OMS workspace (select the Free Tier and put it on the same RG and location) and link it to Traffic Analytics
Traffic Analytics provides rich analytics and visualization derived from NSG flow logs and other Azure resources' data. Drill through geo-map, easily figure out traffic hotspots and get insights into optimization possibilities. Learn about all features To use this feature, choose an OMS workspace. To minimize data egress costs, we recommend that you choose a workspace in the same region your flow logs storage account is located. Network Performance Monitor solution will be installed on the workspace. We also advise that you use the same workspace for all NSGs as much as possible. Additional meta-data is added to your flow logs data, to provide enhanced analytics.

![flow log ta](/images/flow-logs-and-traffic-analytics.PNG)

Click Save

![NSG and TA enabled](/images/NSG-and-TA-enabled.PNG)

Finally, to go to Network Watcher, on the left-side of the portal select All services, then enter Monitor in the Filter box. When Monitor appears in the search results, select it. To start exploring traffic analytics and its capabilities, select Network watcher, then Traffic Analytics
The dashboard may take up to 30 minutes to appear the first time because Traffic Analytics must first aggregate enough data for it to derive meaningful insights, before it can generate any reports.

### 6.  Lab 3 – Control outbound security traffic with Azure Firewall

Azure Firewall is a stateful firewall as a service, built in with high availability and cloud scalability. The primary use case for the Azure Firewall is to centrally create, enforce and log application and network policies. As the first version of the product, the firewall is focused on securing outbound flows by FQDN whitelisting/blacklisting. It provides source network address translation and it’s integrated with Azure Monitor for logging and analytics

At this time the Azure Firewall is on public preview. To enable the firewall in your subscription you need use the following Azure PowerShell commands:

```
Register-AzureRmProviderFeature -FeatureName AllowRegionalGatewayManagerForSecureGateway -ProviderNamespace Microsoft.Network
```

```
Register-AzureRmProviderFeature -FeatureName AllowAzureFirewall -ProviderNamespace Microsoft.Network
```

It takes up to 30 minutes for feature registration to complete. You can check your registration status by running the following Azure PowerShell commands:

```
Get-AzureRmProviderFeature -FeatureName AllowRegionalGatewayManagerForSecureGateway -ProviderNamespace Microsoft.Network
```

```
Get-AzureRmProviderFeature -FeatureName AllowAzureFirewall -ProviderNamespace Microsoft.Network
```

After the registration is complete, run the following command:

```
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network
```

The cloud firewall needs to have a subnet within your VNET named ‘AzureFirewallSubnet’

On the portal create a new subnet in your VNET with that name (i.e 10.0.6.0/24)

Go to your resource, click Create a resource, networking, and look for ‘Firewall

![firewall](/images/firewall.PNG)

We will work on the Web VM, and we will change the default route of the web subnet to go through the firewall

**Create a default Route**

![route table](/images/route-table.PNG)

1.	Click Subnets, and then click Associate.
2.	Click Virtual network, and then select ‘ra-ntier-vnet’
3.	For Subnet, click ‘mgmt
4.	Click OK.
5.	Click Routes, and then click Add.
6.	For Route name, type FW-DG.
7.	For Address prefix, type 0.0.0.0/0.
8.	For Next hop type, select Virtual appliance.
    Azure Firewall is actually a managed service, but virtual appliance works in this situation.
9.	For Next hop address, type the private IP address for the firewall 
10.	Click OK.

**Create application rules**

1.	Click on the firewall
2.	Under Settings, click Rules.
3.	Click Add application rule collection.
4.	For Name, type App-Coll01.
5.	For Priority, type 200.
6.	For Action, select Allow.
7.	Under Rules, for Name, type AllowGH.
8.	For Source Addresses, type 10.0.1.0/24
9.	For Protocol:port, type http, https. 
10.	For Target FQDNS, type github.com
11.	Click Add.

*Note:Azure Firewall includes a built-in rule collection for infrastructure FQDNs that are allowed by default. These FQDNs are specific for the platform and can't be used for other purposes. The allowed infrastructure FQDNs include:*
- Compute access to storage Platform Image Repository (PIR).
- Managed disks status storage access.
- Windows Diagnostics
*You can override this build-in infrastructure rule collection by creating a deny all application rule collection which is processed last. It will always be processed before the infrastructure rule collection. Anything not in the infrastructure rule collection is denied by default.*

**Configure network rules**

The idea is to permit DNS traffic to go through the Firewall from a L3/L4 perspective
1.	Click Add network rule collection.
2.	For Name, type Net-Coll01.
3.	For Priority, type 200.
4.	For Action, select Allow.
5.	Under Rules, for Name, type AllowDNS.
6.	For Protocol, select UDP.
7.	For Source Addresses, type 10.0.2.0/24.
8.	For Destination address, type 168.63.129.16
9.	For Destination Ports, type 53.
10.	Click Add.

**Test your firewall rules**

Open a browser and try go to github. 

![firewall allowed](/images/Firewall-allowed-rule.PNG)

Try going to another page, this action should be blocked:

![firewall blocked](/images/Fireall-blocked-rule.PNG)

### 7. Lab 4 – Protecting the Web Application - Application Gateway and WAF (Web Application Firewall)

Access to the Web VMs will be through an Application Gateway, which acts as a Layer7 HTTP reverse proxy and can load balance the web traffic. The Application Gateway can be enabled with a WAF to protect our application against know vulnerabilities like the OWASP Top 10. Internet access to the Web tier should go through the AppGW and we will associate a public IP address to the gateway

Run through Powershell, Azure CLI or an ARM template for the creation of the AppGW/WAF:

Azure ARM template:  on https://github.com/davisanc/AzureSecurityLabs , download the app_gw-security-labs.json and deploy as an ARM template

Azure CLI:
```
az network application-gateway create  --name myAppGateway --location uksouth --resource-group Sec-Foundation-MTC --capacity 2 --sku Standard_Medium --http-settings-cookie-based-affinity Disabled --public-ip-address myAGPublicIPAddress  --vnet-name ra-ntier-vnet --subnet appgateway   --servers 10.0.1.5
```
**Optional: Create a (DVWA) Damn Vulnerable Web Application from the marketplace, add the VM to the Application Gateway Pool and run a sequence of SQL injection and XSS attacks, checking the WAF can stop them**
This DVWA is managed by a 3rd party company so the Azure pass credits cannot be used. You will need to use an enterprise subscription or a Pay-as-You-Go

![DVWA](/images/dvwa-vm.PNG)

### 8. Lab 5 – Understand your application security posture in Azure -Azure Security Center for security recommendations

First thing is to upgrade your subscription to the Standard Tier to get all features of ASC

**Enable your Azure subscription**

1.	Sign into the Azure portal.
2.	On the Microsoft Azure menu, select Security Center. Security Center - Overview opens.

![oms](/images/OMS.png)

**Security Center – Overview** provides a unified view into the security posture of your hybrid cloud workloads, enabling you to discover and assess the security of your workloads and to identify and mitigate risk. Security Center automatically enables any of your Azure subscriptions not previously onboarded by you or another subscription user to the Free tier.

You can view and filter the list of subscriptions by clicking the Subscriptions menu item. Security Center will now begin assessing the security of these subscriptions to identify security vulnerabilities. To customize the types of assessments, you can modify the security policy. A security policy defines the desired configuration of your workloads and helps ensure compliance with company or regulatory security requirements.

Within minutes of launching Security Center the first time, you may see:
- **Recommendations** for ways to improve the security of your Azure subscriptions. Clicking the Recommendations tile will launch a prioritized list.
- An inventory of **Compute & apps, Networking, Data security, and Identity & access** resources that are now being assessed by Security Center along with the security posture of each.

To take full advantage of Security Center, you need to complete the steps below to upgrade to the Standard tier and install the Microsoft Monitoring Agent.

**Upgrade to the Standard tier**

For the purpose of the Security Center quickstarts and tutorials you must upgrade to the Standard tier. Your first 60 days are free, and you can return to the Free tier any time.
1.	Under the Security Center main menu, select **Onboarding to advanced security**
2.	Under Onboarding to advanced security, Security Center lists subscriptions and workspaces eligible for onboarding. Select a subscription from the list.

![oms advanced](/images/oms-advanced.png)




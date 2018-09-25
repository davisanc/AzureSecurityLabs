# Azure Security Workshop - Labs

These series of labs are based on the **[Azure Reference Architecture](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/n-tier/n-tier-sql-server)** to deploy an **N-Tier application**

To simplify and speed up the process of creating this architecture we will only deploy one VM per Tier. Also, the Domain Controller VM 
and AD Domain Services are removed from the lab template, as this could be run on a separate workshop

![image of 3tier](/images/n-tier-sql-server.png)

## Lab Series

1.	Create a Microsoft Account, redeem your Azure Passes and activate your credit
2.	Configuration needed before starting the labs
3.	Deploy the solution
4.	Lab 1 - Protecting the Network Perimeter – NSG (Network Security Groups)
5.	Lab 2 - Azure Networking logs
6.	Lab 3 - Control outbound security traffic with Azure Firewall
7.	Lab 4 - Protecting the Web Application - Application Gateway and WAF (Web Application Firewall)
8.	Lab 5 - Understand your application security posture in Azure - Azure Security Center for security recommendations
9.	Lab 6 - Storage Security – Encryption at Rest - Apply disk encryption to a running VM
10.	Lab 7 - Extending your Data Centre to Azure in a secure way – Site to Site VPN Access
11.	Lab 8 - Azure Active Directory Role-Based Access Control
12.	Lab 9 - Enable DDoS protection for your resources

## 1.	Create a Microsoft Account, redeem your Azure Passes if you will use your Personal Microsoft Account

*Note: We recommend using an Enterprise Subscription with a working account that has privileges to create resources. There are some labs on this guide that require the use of 3rd party Virtual Machines that do not allow the use of Azure Pass credits. In order to minimise costs we will enable ‘auto’shutdown’ of Virtual Machines.* 

If you plan to use an Enterprise account with your enterprise subscription, please go to the next section of the lab series **Configuration needed before starting the labs**

If you want to use the Azure pass provided, please follow these instructions **ABSOLUTELY TO THE LETTER**. Please read each step twice, before you do it. This is eminently important: it’s possible to lock yourself out of your free subscription or create scenarios where the pass “is already in use”, where you get a message saying the “pass is not associated with this account” and about a dozen variants of these basic ideas. If you follow these instructions **TO THE LETTER WITH NO STEPS SKIPPED OR ALTERNATIVE STEPS TAKEN** (because “I can see what he means, but there’s no need to do that – I know what I’m doing”) – then you will have success. It’ll take you less than 5 minutes and you’ll have started the Azure labwork while others are still trying to activate their subscriptions. 

You can only assign one free trial/Azure Pass during the entire life of a Hotmail/outlook.com/Live-ID… Therefore, **avoid using an existing outlook.com/Hotmail.com/LiveID**. We’ve come across it many times where people took out a trial in 2008 and have completely forgotten all about it. You’ll only get an error message saying you can’t do that….

1.	Close down all web browsers
2.	Fire up a single InPrivate instance of Internet Explorer. Double check to make sure you have only a single, InPrivate instance of Internet Explorer loaded.
3.	Go to http://account.live.com. If it shows you are logged in, click the sign-out button, close the browser down and fire up another InPrivate Instance.
4.	Go to http://outlook.com and create a new account…
    
    Use the azsecseptX@outlook.com , where X has given to you by the proctors

![image of outlook](/images/outlook.png)

…or you might get a box like this:

![image of signin](/images/signin.png)

5.	Go through the sign-up process and remember your username and password.
6.	Click the icon in the top right corner of the page:

![image of account](/images/account.png)

7.	Click Sign out on the right hand pane that appears:

![image of signout](/images/signout.png)

8.	Shut down every browser and then open a single InPrivate browser.

9.	Go to http://microsoftazurepass.com

10.	Click the Start button

![image or redeem](/images/redeem-your-azure-pass.PNG)

11.	Enter your credentials and click the Sign in button:

![image of credentials](/images/credentials.png)

12.	Click the **Confirm Microsoft Account** button.

![image of confirm](/images/confirm.png)

13.	Enter the promo code you instructor has given you and then click **Claim Promo Code**

![image of claim](/images/claim.png)

14.	It takes a few minutes for the page to move on. Eventually you end up at an **ACTIVATE** button – click it:

![active pass](/images/activate-azure-pass.PNG)

15.	After about 30 seconds you are asked to fill in a form. Click Next after filling in section 1.

![image of form](/images/form.png)

16.	You are prompted for a phone number enter some data and click Next. It might take a minute or two to process…

![image of phone](/images/phone.png)

17.	At the agreement screen – agree to the terms and conditions. You might opt out of the marketing info – your choice, then click **Sign-Up**:

![image of signup](/images/signup.png)

18.	This part also takes a few minutes. There’s one point where you get a page saying “This typically takes up to 4 minutes”. **BE CAREFUL!** – experience has taught me to never click anything on this page until you see a page saying “Your subscription is ready for you”. 

19.	Click the “**Or get started with your Azure subscription >”** hyperlink. **DO NOT click the green button**:

![image of welcome](/images/welcome.png)

20.	Eventually you end up at a button that says “manage my subscription” – go ahead. You’re ready…
21.	You’re now in your Azure subscription. You’re offered to go on a tour of the UI – it’s probably best to save time and click **Maybe later**
22.	To check whether you genuinely have a subscription and not just a UI that will fail to provision anything, click **Cost Management and Billing** near the bottom of the left-hand pane:

23.	If the **SUBSCRIPTION** and **SUBSCRIPTION ID** columns are blank, something has gone wrong – talk to your instructor. Otherwise, you’re good to go. **Congratulations!**

![image of ready](/images/ready.png)


## 2.  Configuration needed before starting the labs (Time to complete: 15 min)

**1. Working with Visual Studio Code**

- If you already have Visual Studio Code, please open it. If not:

  - Install Visual Studio Code from the [Visual Studio Code website](https://code.visualstudio.com)
  - Allow Visual Studio Code to launch after installation. If you already have Visual Studio Code installed, open it

- In Visual Studio Code, click the **View** menu option and select **Terminal**. This will open a PowerShell terminal command prompt which is a great place to run the Azure CLI commands from in the labs.

**2. Powershell AzureRM module**

- Install the AzureRM module. On your Powershell console:

    ```/PowerShell
    Install-Module AzureRM
    ```

    If you get an error about Administrator rights, either open Visual Studio Code with elevated privileges or run the above command with the `-Scope CurrentUser` parameter.

- Make sure you have installed AzureRM version 6 (or higher).

    ```/PowerShell
    Get-Module AzureRM -ListAvailable | Select-Object -Property Name,Version,Path
    ```

**3. Install Azure CLI 2.0**

- Visit the [Azure CLI 2.0 website](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) and install the Azure CLI onto your computer.

- Open a Command Prompt (or the Terminal Window in Visual Studio Code) and check that running the `az` command produces command help output (you may need to restart Windows Powershell/Visual Studio Code and re-open again for the installation to register).

- *Note: you might face an issue when you try to run an az command that says*

    ```
    az : The term 'az' is not recognized as the name of a cmdlet, function, script file, or operable program.
    ```

    The issue is because the Azure CLI 2.0 is installed in location - C:\Users\<username>\AppData\Local\Programs\Python\Python37-32\Scripts\ which is not added to the PATH variable.*
    1. *First make sure you have python installed in your machine. If you don’t have the original CLI (or python) at all, you need that  first. Download and install it from here: https://www.python.org/downloads/release/python-352/*
    2. Uninstall Azure CLI earlier versions with command - pip uninstall azure-cli
    3. Re-install Azure CLI 2.0 - pip install --user azure-cli
    4. Add the path C:\Users\<username>\AppData\Local\Programs\Python\Python37-32\Scripts\ to PATH
    5. Check if the az command is working:

        ```
        az --help
        ```

**4. (Optional) Install Visual Studio Code Extensions**
In Visual Studio Code, go to Extensions, search for **Azure CLI Tools** and install the package. Reload Visual Studio Code once installed.

**5. Install the [Azure building blocks npm package](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks)**

- Install **Node.js** from the link above. The LTS version will be fine.
- You may need to close and re-open PowerShell/Visual Studio Code/Terminal window after the Node.js installation for the following commands to work.
- Install the Azure Building Blocks package:

   ```
   npm install -g @mspnp/azure-building-blocks
   ```

- Test Azure Building Blocks with the following command on PowerShell or VS Code:
   ```
   azbb
   ```

**6. From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:**
  ```
  az login
  ```

**7. Set the CLI to use the correct subscription (Enterprise or Azure Pass)**
  To get your Subscription ID, run the following command to list the Subscriptions within your account:
  ```
  az account list --query "[].{id:id,name:name}" --output tsv
  ```

  Make a note of the relevant ID for your Subscription, and set the CLI to use this as the active subscription:
  ```
  az account set --subscription  "<subscription-ID>"
  ```

## 3.  Deploy the Solution (Time to complete: 35 to 40 min)

1. Run the following command to create a Resource Group

    **VERY IMPORTANT: Use location 'WestEurope' to deploy your infrastructure for all labs. This will guarantee that you won't have issues with the Azure Pass credits and availability of Virtual Machine types.**

    ```
    az group create --location <location> --name <resource-group-name>
    ```

2. Run the following command to create a Storage Account for your Cloud resources.
    *Note: Storage account name must be between 3 and 24 characters in length and use numbers and lower-case letters only*

    ```
    az storage account create --location <location> --name <storage-account-name> --resource-group <resource-group-name> --sku Standard_LRS
    ```

3. In your browser, navigate to https://github.com/davisanc/AzureSecurityLabs 

4. Open the **n-tier-windows-security-labs.json** file. This file is an Azure Resource Manager (ARM) Template which describes the infrastructure resources we need to use for this lab as code. When deployed, the template will instruct Resource Manager to create the resources as described in the text.

    For more on ARM Templates, please visit: 
    [https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#template-deployment](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#template-deployment)

5. Just above the main text pane on the Github page, click the **Raw** button to see the unformatted JSON.

6. Copy and paste the text into a new file in Visual Studio Code. In the JSON text, search for all instances of **adminUsername** and **adminPassword** and replace both the property values with your own admin username and passwords, and save the file locally as **n-tier-windows-security-labs.json**.

    Example:

    Before...
    ```
    "adminUsername": "admin",
    "adminPassword": "P4ssw0rd123$%",
    ```
    After...
    ```
    "adminUsername": "adminUserName",
    "adminPassword": "C4ndY*Fl0$S18",
    ```

7. Run the following `azbb` command to deploy the base resources required for the lab using the ARM template modified above. This will include a Jumpbox, Web VM, Application VM and a SQL Server VM:

    ```
    azbb -s <Subscription-ID> -g <Resource-Group-Name> -l <Location> -p .\<name-of-your-tempalte.json> --deploy
    ```

    *Note: this environment will need about 35 minutes to deploy. Once the last command completes, report to your proctors that you have reached this point*

    The Application and Database tier should have an internal Load Balancer in front of them as per the Azure Reference Architecture, so you can scale up the tier with more VMs if needed and the Load Balancer will distribute the traffic accordingly. However, to speed up the process of creating this architecture, there will be no Load Balancers at the Application and Database Tiers

    The Web Tier is not currently load balanced, as we will create an external Application Gateway in front of the web tier later in the labs.

    For now, the only Internet access to the environment is through the Jump Box as it is the only VM with a Public IP address.

    **Test: make sure you can ping from the JB to the Web, Biz and DB virtual machines (enable PING on the firewall settings)**

## 4.  Lab 1 - Protecting the Network Perimeter with Network Security Groups

Network Security Groups filter traffic to and from resources in an Azure virtual network. They can be applied at subnet or virtual machine level, and filter the traffic based on a set of rules.

Further information on network security groups (NSG) can be found in the Azure documentation:

[https://docs.microsoft.com/en-us/azure/virtual-network/security-overview](https://docs.microsoft.com/en-us/azure/virtual-network/security-overview)

You can use security groups to protect/restrict traffic between tiers. In the 3-tier architecture shown, the web tier should not communicate directly with any resource in the database tier. To enforce this, the database subnet should block all incoming traffic from the web tier subnet. This can be done using a security group.

### 4.1 - Creating the security group
This section creates the security group to protect the database tier.

1.  In the VS Code terminal, enter the following CLI command to create the security group **SQL-NSG**
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

3.	Allow inbound traffic from the database tier subnet itself.

    This rule allows communication between the database VMs, which is needed for database replication and failover. Use the database (SQL) subnet range 10.0.3.0/24 as the source:

    ```
    az network nsg rule create --name AllowFromSQL --nsg-name SQL-NSG --priority 120 --resource-group <resource-group-name> --description "Allow all intra SQL traffic within the Database Tier" --access Allow --direction Inbound --source-address-prefix 10.0.3.0/24 --source-port-ranges * --protocol * --destination-address-prefix * --destination-port-ranges *
    ```

4.	Allow RDP access from the Jump Box.

    Allowing RDP traffic on the RDP port (3389) from the Jump Box lets remote administrators connect to the database servers from the Jump Box. By only specifying the RDP port, users of the Jump Box will not be able to connect via any other method, port or protocol.

    Use the management subnet range 10.0.0.128/25 as the source:

    ```
    az network nsg rule create --name AllowRDPFromJB --nsg-name SQL-NSG --priority 130 --resource-group <resource-group-name> --description "Allow RDP traffic from the Jump Box" --access Allow --direction Inbound --source-address-prefix 10.0.128.0/25 --source-port-ranges * --protocol TCP --destination-address-prefix * --destination-port-ranges 3389
    ```

5.	Deny all other inbound traffic from the VNet.

    Now we have set up the base access requirements, we need to block all other traffic from within the VNet. Instead of a source address, we can use the **VirtualNetwork** tag in the rule:

    ```
    az network nsg rule create --name DenyFromVNet --nsg-name SQL-NSG --priority 140 --resource-group <resource-group-name> --description "Deny general VNet traffic" --access Deny --direction Inbound --source-address-prefix VirtualNetwork --destination-port-ranges *
    ```

6.	Deny all inbound traffic from the Internet.

    Create the final rule yourself using the commands above as the pattern. Set your own description, and use these properties:

    - **Name:** DenyFromInternet
    - **Priority:** 150
    - **Source Address:** Internet

#### Rule Priority
Consider the following when creating security group rules...

- Security group rules run in priority order, with the lowest priority rule being evaluated first.
- Leave a reasonable gap between your rules. It makes for a lot of work to try and retro-fit a new rule with a higher priority in between rules priorties 4,5 and 6 than 140, 150 and 160.
- The first **Deny** rule encountered by the evaluation instantly deny the access.

### 4.2 - Attach the security group to the SQL Server network interface / NIC

Follow these steps to attach the new NSG to the network interface of the SQL VM...

![NSG-inbound-sql](/images/attach-NSG-NIC.PNG)

Alternatively you can use the following CLI command to make this attachment. The command references the NIC resource and attaches the security group to the NIC.

**Note:** The name of the NIC for the SQL server is **sql-vm1-nic1**

```
az network nic update --resource-group <resource-group-name> --name sql-vm1-nic1 --network-security-group SQL-NSG
```

### 4.3 - Overview and final steps

Your NSG rule set should look similar to this...

![NSG-inbound-sql](/images/NSG-SQL-NIC.PNG)

Confirm that you can RDP from the Jump Box to the SQL server and also from the Business VM but not from the Web VM (you will need to RDP to the Jump Box first, then RDP to the Business and Web VMs to finally RDP to the SQL VM)

![RDP blocked](/images/RDP-blocked-from-web.PNG)

## 5. Lab 2 - Azure networking logs

Network logging and monitoring in Azure is comprehensive and covers two broad categories:
- **Network Watcher**: Scenario-based network monitoring is provided with the features in Network Watcher. This service includes packet capture, next hop, IP flow verify, security group view and NSG flow logs. Scenario level monitoring provides an end to end view of network resources in contrast to individual network resource monitoring.
- **Resource monitoring**: Resource level monitoring comprises four features: diagnostics logs, metrics, troubleshooting, and resource health. All of these features are built at the network resource level.

To troubleshoot your NSG rules, enable NSG flow logs. This will enable Network watcher. 
Go to the search bar on the Portal, look for Network Watcher. Filter by your Subscription ID and Resource Group.

![network watcher](/images/Network-watcher.PNG)

Click on the NSG you worked on on the previous lab, enable **Flow Logs**

You will need to select an storage account within your Resource Group. The NSG flow logs will be stored within a blob container of the selected storage account. More details https://docs.microsoft.com/en-us/azure/network-watcher/network-watcher-nsg-flow-logging-portal

![NSG flow logs](/images/NSG-flow-logs-enabled.PNG)

Also, you may want to enable **Traffic Analytics for rich analytics and visualization**. You will need to create an **OMS workspace** (select the Free Tier and put it on the same RG and location) and link it to Traffic Analytics
Traffic Analytics provides rich analytics and visualization derived from NSG flow logs and other Azure resources' data. Drill through geo-map, easily figure out traffic hotspots and get insights into optimization possibilities. Learn about all features To use this feature, choose an OMS workspace. To minimize data egress costs, we recommend that you choose a workspace in the same region your flow logs storage account is located. Network Performance Monitor solution will be installed on the workspace. We also advise that you use the same workspace for all NSGs as much as possible. Additional meta-data is added to your flow logs data, to provide enhanced analytics.

![flow log ta](/images/flow-logs-and-traffic-analytics.PNG)

Click Save

![NSG and TA enabled](/images/NSG-and-TA-enabled.PNG)

Finally, to go to Network Watcher. On the left-side of the portal select All services, then enter **Monitor** in the Filter box. When Monitor appears in the search results, select it. To start exploring traffic analytics and its capabilities, select Network watcher, then Traffic Analytics
The dashboard may take up to 30 minutes to appear the first time because Traffic Analytics must first aggregate enough data for it to derive meaningful insights, before it can generate any reports.
Also, you may get the following message: *Displaying only resources data. No flow information is available for the selected workspace. Please refresh and load the workspace again after some time to see flow logs data.*

![TOPOLOGY](/images/traffc-analytics.PNG)

Also, you may want to visualize the Network Topology. In Network Watcher, click on **Topology** (Filter by Subscription, RG and VNET)

![TOPOLOGY](/images/topology.PNG)

## 6.  Lab 3 – Control outbound security traffic with Azure Firewall

Azure Firewall is a stateful firewall as a service, with high availability and cloud scalability built-in. The primary use case for the Azure Firewall is to centrally create, enforce and log application and network policies. As the **first version** of the product, the firewall is focused on **securing outbound flows by FQDN whitelisting/blacklisting**. It provides source network address translation and it is integrated with Azure Monitor for logging and analytics.

### 6.1 - Enabling Azure Firewall preview

The Azure Firewall is currently in public preview. To enable the firewall in your subscription you need use the following Azure PowerShell commands:

To enable the firewall in your subscription you need use the following Azure PowerShell commands:

1. Run this command from PS
```
Register-AzureRmProviderFeature -FeatureName AllowRegionalGatewayManagerForSecureGateway -ProviderNamespace Microsoft.Network
```

*Note: If you get the following error:*
    
```
Powershell error – Import-Module : File AzureRM.psm1 cannot be loaded because running scripts is disabled on this system
```
*Run this command:*
```
PS c:\> Set-ExecutionPolicy RemoteSigned
```
You may need to login again to your Azure subscription with **Connect-AzureRmAccount** or **az login**. Make sure you are on the right subscription
    
2. Now run the following PS command needed to enable the Firewall
```
Register-AzureRmProviderFeature -FeatureName AllowAzureFirewall -ProviderNamespace Microsoft.Network
```

**It takes up to 30 minutes for feature registration to complete**. You can check your registration status by running the following Azure PowerShell commands:

```
Get-AzureRmProviderFeature -FeatureName AllowRegionalGatewayManagerForSecureGateway -ProviderNamespace Microsoft.Network
```

```
Get-AzureRmProviderFeature -FeatureName AllowAzureFirewall -ProviderNamespace Microsoft.Network
```

After the registration is complete, run the following PowerShell command:

```
Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Network
```

### 6.2 - Create a subnet for the firewall

The cloud firewall needs to have a dedicated subnet within your VNet named **AzureFirewallSubnet**. 

On the portal create a new subnet in the **ra-ntier-vnet** VNet named **AzureFirewallSubnet** with the IP address space of **10.0.6.0/24**.

Alternatively, run the following CLI command to create the subnet:
```
az network vnet subnet create --address-prefix 10.0.6.0/24 --name AzureFirewallSubnet --resource-group <resource-group-name> --vnet-name ra-ntier-vnet
```

### 6.3 - Create the firewall

On the Azure Portal, click **Create a resource**, **Networking**, and look for **Firewall**. Configure the firewall as per the image below...

![firewall](/images/firewall.PNG)

Once the firewall object is created, view the properties of the firewall and take a note of the private IP address.

### 6.4 Routing the web-tier traffic
We will work on the Web VM, and we will set the default route of the web-tier subnet to send all traffic through the firewall.

1. Create a new Route Table

    ![route table](/images/route-table.PNG)

    The Azure CLI command to create the route table is:

    ```
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

    ```
    az network route-table route create --resource-group <resource-group-name> --route-table-name Firewall-Route --name FW-DG --address-prefix 0.0.0.0/0 --next-hop-type VirtualAppliance --next-hop-ip-address <firewall-ip-address>
    ```

3. Attach the route table to the web subnet

    1. Click **Subnets** in the list options for the route table.
    2. Click **Associate**.
    3. Click **Virtual network** and select **ra-ntier-vnet** from the list of VNets.
    4. Choose the **web** subnet from the list given, and click **OK**.

    To achieve this using the Azure CLI:

    ```
    az network vnet subnet update --name web --resource-group <resource-group-name> --vnet-name ra-ntier-vnet --route-table Firewall-Route
    ```

    You may notice that the CLI method attaches the route table to the subnet inside the VNet using the 'network vnet' command subset, rather than setting this as a property on the route table object itself.

### 6.5 - Create application rules

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

### 6.6 - Configure network rules

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

### 6.7 - Test your firewall rules

RDP to the JumpBox and from there to the Web VM. Open a browser and try go to github.com.

![firewall allowed](/images/Firewall-allowed-rule.PNG)

Try going to another site, this action should be blocked:

![firewall blocked](/images/Fireall-blocked-rule.PNG)

![bbc blocked](/images/bbc.PNG)

## 7. Lab 4 – Protecting the Web Application - Application Gateway and Web Application Firewall (WAF)

In this architecture, access to the web services running on the web VMs will be via an Application Gateway, which acts as a Layer7 HTTP reverse proxy and can load balance web traffic. The Application Gateway can be enabled with a WAF to protect our application against known vulnerabilities, such as those listed on the the [2017 OWASP Top 10 list](https://www.owasp.org/index.php/Top_10-2017_Top_10).

Internet traffic destined for the web servers should always go through a load balancer or Application Gateway where possible, with the advertised endpoint for the traffic set as a public IP address attached to the gateway.

### 7.1 - Creating the Application Gateway

#### Using ARM Templates

We have included a json file to deploy the application gateway as an ARM template. The template, **app_gw-security-labs.json** is listed above in this repository. Click on **Raw** , copy the content of the file and paste it to a new file in Visual Studio Code. Save it as a json file with the same name

To deploy the template, on the Azure Portal go to **Create a resource** at the left panel, search for **Template Deployment**, click **Create**, and then click **Build your own template in the editor**

![image of template](/images/template.PNG)

Click on **Load File** and use the json file you have just saved to your PC. Click on **Save**

Fill in the **BASICS** part with the Resource Group name and location, also the **SETTINGS** area with the VNET name **ra-ntier-vnet** and **appgateway** subnet (The Application Gateway sits on its own subnet). 

You will notice the template has already the IP address value of the Backend server (the Web VM with IIS)

The template will deploy the AppGW with **WAF enabled on Detection mode**

Click on **Purchase**

![image of purchase template](/images/purchase-template.PNG)

The creation of the Application Gateway will take a few minutes

#### Azure CLI

This command creates the Application Gateway (for the purposes of this lab) into the **appgateway** subnet of our VNet. Please review the Azure CLI documentation for [creating an application gateway](https://docs.microsoft.com/en-us/cli/azure/network/application-gateway?view=azure-cli-latest#az-network-application-gateway-create) to see the full list of possible parameters.

```
az network application-gateway create --name myAppGateway --location <location> --resource-group <resource-group-name> --capacity 2 --sku Standard_Medium --http-settings-cookie-based-affinity Disabled --vnet-name ra-ntier-vnet --subnet appgateway --servers <web-server-ip-address>
```

**Please note:** the IP address of the web server required is the *internal* IP address of web server **web-vm1**. You can get the address under the **Networking** settings for the resource or by running the following CLI command:

```
az network nic show --name web-vm1-nic1 --resource-group <resource-group-name> --query "ipConfigurations[].privateIpAddress"
```

**Please also note** that we did not specify Public IP Address resources for this application gateway. Not specifying an IP address will create a new Public IP address, however you can be more descriptive and create these first before creating the gateway.

### 7.2 - Test the Application Gateway

Go to the AppGW resource you just created, and look for the Public IP address assigned under **Frontend public IP address**. Open a new browser tab and introduce the Public IP address, and confirm that you can reach to the Web VM IIS page through the AppGW

### 7.3 - (Optional) - Test the Azure WAF with a DVWA

We will create a DVWA (Damn Vulnerable Web Application) VM from the marketplace, add the VM to the Application Gateway Pool and run a sequence of SQL injection and XSS attacks to test that the WAF in detection and blocking mode

**IMPORTANT:** This DVWA is managed by a 3rd party company, so Azure Pass credits cannot be used. You will need to use either an Enterprise or Pay-as-You-Go subscription.

If you are interested to test the Azure WAF or any other 3rd party WAFs using the DVWA, please report to your proctors for addional guidance and further steps to simulate some attacks like SQL injection, XSS, etc

![DVWA](/images/dvwa-vm.PNG)

## 8. Lab 5 – Understand your application security posture in Azure -Azure Security Center for security recommendations

First thing is to upgrade your subscription to the Standard Tier to get all features of Azure Security Center

**Enable your Azure subscription**

1. Sign into the Azure portal.
2. On the Microsoft Azure menu, select **Security Center**. The **Security Center - Overview** opens.

![oms](/images/OMS.png)

**Security Center – Overview** provides a unified view into the security posture of your hybrid cloud workloads, enabling you to discover and assess the security of your workloads and to identify and mitigate risk. Security Center automatically enables any of your Azure subscriptions not previously onboarded by you or another subscription user to the Free tier.

You can view and filter the list of subscriptions by clicking the Subscriptions menu item. Security Center will now begin assessing the security of these subscriptions to identify security vulnerabilities. To customize the types of assessments, you can modify the security policy. A security policy defines the desired configuration of your workloads and helps ensure compliance with company or regulatory security requirements.

Within minutes of launching Security Center the first time, you may see:
- **Recommendations** for ways to improve the security of your Azure subscriptions. Clicking the Recommendations tile will launch a prioritized list.
- An inventory of **Compute & apps, Networking, Data security, and Identity & Access** resources that are now being assessed by Security Center along with the security posture of each.

To take full advantage of Security Center, you need to complete the steps below to upgrade to the Standard tier and install the Microsoft Monitoring Agent.

**Upgrade to the Standard tier**

For the purpose of the Security Center quickstarts and tutorials **you must upgrade to the Standard tier**. Your first 60 days are free, and you can return to the Free tier any time.

1. Under the Security Center main menu, select **Getting started**
2. Security Center lists subscriptions eligible for onboarding. Under **Apply your trial on 1 subscription**, select yours from the list (Azure Pass or your enterprise subscription). Click on **Start Trial**

    ![asc upgrade](/images/ASC-upgrade.PNG)
    
3. To make the most of Azure Security Center, we need to enable **data collection agents**. They must be installed on your virtual machines for data collection so we can receive security alerts and recommendations

We will install agents automatically. Install the **ASC agents** to your subscription

![image of agents](/images/install-agent.PNG)
    
Now that you’ve upgraded to the Standard tier, you have access to additional Security Center features, including **adaptive application controls, just in time VM access, security alerts, threat intelligence, automation playbooks**, and more. Note that security alerts will only appear when Security Center detects malicious activity.

![oms global](/images/oms-global.png)

**Recommendations**

With this new insight into your Azure VMs, Security Center can provide additional recommendations related to system update status, Operating System security configurations, endpoint protection, as well as generate additional security alerts.

![oms recomm](/images/asc-recommendations.PNG)

## 9. Lab 6 - Storage Security – Encryption at Rest - Apply disk encryption to a running VM

Having looked at Azure Security Center we can see recommendations to apply disk encryption to our VMs. This can be done with the help of Azure Key Vault.

Full details about Azure Key Vault can be found here: [https://azure.microsoft.com/en-us/services/key-vault/](https://azure.microsoft.com/en-us/services/key-vault/)

For Azure Disk encryption to work, the Key Vault and the VMs must be co-located in the same Azure region and subscription.

### 9.1 - Create the Azure Key Vault

Create a new Key Vault from the portal following the steps in the image below, or create a Key Vault using this CLI command:

```
az keyvault create --name <your-keyvault-name> --resource-group <resource-group-name> --sku standard
```

![create keyv](/images/Create-KeyVault.PNG)

### 9.2 - Prerequisites to go before running encryption

You can enable encryption by using a template, PowerShell cmdlets, or CLI commands. The following sections explain in detail how to enable Azure Disk Encryption.

*Important: It is mandatory to snapshot and/or backup a managed disk based VM instance outside of, and prior to enabling Azure Disk Encryption. A snapshot of the managed disk can be taken from the portal, or Azure Backup can be used. Backups ensure that a recovery option is possible in the case of any unexpected failure during encryption.*

There are some prerequistes to check before enabling disk encryption. They can be found here:
https://docs.microsoft.com/en-us/azure/security/azure-security-disk-encryption-prerequisites

We have summarized them for you here. Alternatively, you can run a pre-requisite script that will run all the following comamnds for you

If you prefer to do this one by one, go through the following steps. If you prefer to go with the pre-req scritpt, jumpt to 9.4 of this lab

**On Powershell**

1. Make sure you have AzureRM module version 6 installed on your local machine.
2. Verify the installed versions of the AzureRM module. The AzureRM module version needs to be 6.0.0 or higher.

    ```
    Get-Module AzureRM -ListAvailable | Select-Object -Property Name,Version,Path
    ```
3. If needed, update the Azure PowerShell module. Using the latest AzureRM module version is recommended (This will require administrative rights to the PowerShell session).

    ```
    Update-Module -Name AzureRM
    ```
4. Install the Azure Active Directory PowerShell module

    ```
    Install-Module AzureAD
    ```

5. Verify the installed versions of the module

    ```
    Get-Module AzureAD -ListAvailable | Select-Object -Property Name,Version,Path
    ```
6. Set up an Azure AD app and service principal
    
    On Az CLI:
    
    ```
    az ad sp create-for-rbac --name "ServicePrincipalName" --password "My-AAD-client-secret" --skip-assignment
    ```
    *Note:The appId returned is the Azure AD ClientID used in other commands. It's also the SPN you'll use for az keyvault set-policy. The password is the client secret that you should use later to enable Azure Disk Encryption. Safeguard the Azure AD client secret appropriately.*
    
7. Set the key vault access policy for the Azure AD app with Azure CLI

    ```
    az keyvault set-policy --name "MySecureVault" --spn "<spn created with CLI/the Azure AD ClientID>" --key-permissions wrapKey -- secret-permissions set
    ```
8. Set key vault advanced access policies

The Azure platform needs access to the encryption keys or secrets in your key vault to make them available to the VM for booting and decrypting the volumes. Enable disk encryption on the key vault or deployments will fail.

Set key vault advanced access policies through the Azure portal
Select your keyvault, go to Access Policies, and Click to show advanced access policies.
Select the box labeled Enable access to Azure Disk Encryption for volume encryption.
Select Enable access to Azure Virtual Machines for deployment and/or Enable Access to Azure Resource Manager for template deployment, if needed.
Click Save.

![image of key vault advanced](/images/keyvault-advancedset.PNG)


### 9.3 Enable encryption on existing or running VMs with Azure CLI**


**Azure CLI**

Encrypt a running VM using a client secret:

```
az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --aad-client-id "<my spn created with CLI/my Azure AD ClientID>"  --aad-client-secret "My-AAD-client-secret" --disk-encryption-keyvault "MySecureVault" --volume-type [All|OS|Data]
```
Powershell pre-req script: you can download the 'DiskEncryption.ps' file available here

**Powershell**
```
$rgName = 'MySecureRg';
$vmName = ‘MyExtraSecureVM’;
$aadClientID = 'My-AAD-client-ID';
$aadClientSecret = 'My-AAD-client-secret';
$KeyVaultName = 'MySecureVault';
$keyEncryptionKeyName = 'MyKeyEncryptionKey';
$KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName $rgname;
$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
$KeyVaultResourceId = $KeyVault.ResourceId;
$keyEncryptionKeyUrl = (Get-AzureKeyVaultKey -VaultName $KeyVaultName -Name $keyEncryptionKeyName).Key.kid;

Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -AadClientID $aadClientID -AadClientSecret $aadClientSecret -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -KeyEncryptionKeyUrl $keyEncryptionKeyUrl -KeyEncryptionKeyVaultId $KeyVaultResourceId;
```
*Note: you can also run disk encryption with key encryption key (out of scope of this lab)*

**Finally, verify the disks are encrypted**
```
az vm encryption show --name "MySecureVM" --resource-group "MySecureRg"
```

![image od disk-enc](/images/disk-enc.PNG)


### 9.4 Use the pre-req script

You can also use the pre-req script, that runs all the previous commands for you
Use the AzureDiskEncryptionPreRequisiteSetup.ps1 on this repository, and run it on Powershell

This shoudd be the final result of the script

![sql vm encrypted](/images/sql-vm-encrypted.PNG)

Verify the disks are encrypted: To check on the encryption status of a IaaS VM, use the Get-AzureRmVmDiskEncryptionStatus cmdlet

```
Get-AzureRmVmDiskEncryptionStatus -ResourceGroupName <resource-group-name> -VMName <vm-name>

```

## 10. Lab 7 - Extending your Data Centre to Azure in a secure way – Site to Site VPN Access

**Note:** Only run this lab if you are using an Enterprise subscription or Pay-As-You-Go, **BUT NOT** the Azure Pass. As the pfSense image is managed by a 3rd company, there is a charge for this which cannot be covered by the Azure Pass credits.

**Create a Firewall for the on-premise VPN side**

We will use a Virtual Machine running pfSense Firewall (open Source) to simulate an onpremise data center. In the Azure Portal, create a new resource within the same Resource Group but in another location (i.e West Europe). Search the Marketplace for the **Netgate pfSense Firewall** as shown below...

![pfsense](/images/pfSense.PNG)

Select a small VM size (i.e B1ms or B2s) using standard HDD and managed disks, create a new VNet and use non-overlapping IP addresses (i.e 192.168.0.0/16 for the address space and 192.168.1.0/24 for the ‘default’ subnet). Assign a new public IP address to the VM, leave the pre-configured NSG (HTTP and HTTPs should be open) and enable ‘auto-shutdown’.

![pfsenseB1](/images/pfSense-B1ms-cost.PNG)

Once created, locate and make a note of the created Public IP address. Open a browser tab and enter the public IP address assigned. Login with the username and password you specified during the creation of the VM.

Give a hostname and a Domain name to your pfSense Virtual Machine.

![pfsense general](/images/pfSense-general.PNG)

Leave the default NTP settings.

By default this pfSense firewall comes with a single NIC. During the installation of the settings it will ask for the WAN ip address, choose DHCP. The IP address assigned to the Firewall is the same as the one you can see on the portal (192.168.1.4).

**Create a VPN Gateway for the Azure VPN side**

In the Azure Portal, bring up the properties of the existing Security Workshop VNet, **ra-ntier-vnet**. Click **Subnets** in the left-hand settings panel, and then click **+ Gateway subnet** to create a Gateway Subnet...

![vpn gw subnet](/images/VPN-GW-subnet.PNG)

The Azure CLI command to create this subnet is as follows:

```
az network vnet subnet create --resource-group <resource-group-name> --vnet-name ra-ntier-vnet --name GatewaySubnet --address-prefix 10.0.7.0/24
```

**Note** that when you create the Gateway subnet, even if you use the CLI command, the Azure Portal will disable the **Gateway subnet** option.

Next, we will create the Virtual Network Gateway in the **ra-ntier-vnet** VNet. In the Azure Portal, create a new resource and search for **Virtual Network Gateway** from the Marketplace. Run through the creation steps taking note of the following settings:

- Set the **Gateway type** as **VPN**
- The VPN type will be **Route-based**
- Choose to create a new Public IP address
- Also, we will use BGP to exchange routes between Azure and the pfSense firewall, so we need to enable the BGP option when creating the gateway. Use a private BGP ASN of 65515.
- Use **UK South** as the location for the gateway.

*Note: it will take a few minutes to create the VPN Gateway*

![create vpn gateway](/images/create-vpn-gateway.png)

Creating the Virtual Network Gateway is also possible using the CLI. The CLI command requires a Public IP Address as a parameter (whereas the Portal will create on on the fly) so this is a two step process.

First, create the Public IP Address resource. We will use the **Basic** SKU which will allocate a Dynamic IP address:

```
az network public-ip create --resource-group <resource-group-name> --name <ip-address-name> --sku Basic --location <location>
```

Next, we can create the Virtual Network Gateway using this IP address as a parameter:

```
az network vnet-gateway create --resource-group <resource-group-name> --name vpnGW-pfSense --vnet ra-ntier-vnet --gateway-type Vpn --vpn-type RouteBased --public-ip-address <vpn-ip-address-name> --sku VpnGw1 --asn 65501 --location <location>
```

Once created, you will find the BGP peer address on your VPN Gateway. This is the local address that BGP will use in your Azure VPN Gateway to initiate a BGP connection to your home gateway.

![VPN-GW-LOCAL](/images/VPN-GW-local-addr.PNG)

Now we are going to create the Local Network Gateway. Azure refers to the VPN device that sits in your onpremise network. You will need to indicate the BGP peer address, your local network behind the Firewall (or local VPN gateway) and a Private BGP ASN (I am using 65501 on the pfSense side). 

Go to your RG, click ‘Add resource’ and look for ‘ Local Network Gateway’. Use the ‘onpremise’ public ip address (example below uses 1.2.3.4)

![local nt gateway](/images/local-nt-gateway.png)

Once the local gateway is created we will **define a connection to our onpremise VPN Gateway**. We will use a private shared key to enable the IPSEC VPN to come up. Remember to mark BGP to ‘enabled’ on your Connection. 

**Configure the pfSense VPN Firewall**

Now, moving to the other end we will use the Web UI on the pfSense firewall to work on the Rules and VPN settings 

To configure a new tunnel, a new Phase 1 IPSEC VPN must be created. Remote Gateway will be the public IP address assigned to your Virtual Network Gateway in Azure. Leave ‘auto’ as IKE key exchange version, selecting WAN as the interface to run the VPN. For the authentication part, use the Pre-Shared Key you have defined. Use the encryption algorithm you need, in my case AES (256 bits), DH group and Hashing algorithm

![ipsec tunnels](/images/ipsec-tunnels.png)

We will then move to Phase 2. This phase is what builds the actual tunnel, sets the protocol to use, and sets the length of time to keep the tunnel up when there is no traffic. For remote network, use the VNET address space. Local subnet will be the address space on the LAN side of the pFsense

Apply changes to take them effect

![pfsense VPN2](/images/pfsense-VPN-P2.PNG)

On pfSense, go to Status, IPSEC, Overview and click **‘Connect VPN’**

*NOTE: When using BGP over a Routed IPSEC tunnel, it wouldn’t be needed the configuration and management of P2 entries. You will me managing routes in the BGP config instead of P2 entries. Routed IPSEC is a pfSense feature available in 2.4.4, my setup runs on 2.4.3 so I will create the P2 entries manually*

**VERY IMPORTANT:** On the local gateway connections, as we are using BGP, **don’t forget to enable BGP or the IPSEC tunnel won’t come up**

![image of forget bgp](/images/forget-bgp.PNG)


**Enable IPSEC traffic on the Virtual Machine pfSense NSG**

Configure a Rule to allow UDP 4500 ((IPsec NAT-T) & 500 (ISAKMP) ports 

**Enable IPSEC traffic through WAN interface of pfSense**

Configure a Rule to allow UDP 4500 ((IPsec NAT-T) & 500 (ISAKMP) ports 

![udp ports IPSEC](/images/UDP-ports-IPSEC-open.PNG)

After we added the relevant rules on the NSG and pfSense WAN interface, the connection is up and running 

![pfsense connections](/images/pfsense-connections.png)

On the pfSense side:

![image of vpn connected](/images/pfsense-connected.PNG)

**Enable BGP traffic through IPSEC interface**

Go to Status, System Logs, Firewall --> you can enable BGP to pass through the IPSEC interface using the logs

![pfsense BGP rule added](/images/pfSense-BGP-rule-added.PNG)

Finally, simply check that you have connectivity from the pfSense Gateway to the Web VM

![telnet port 80](/images/telnet-port-80-to-web.PNG)

## 11. Lab 8 - Azure Active Directory Role-Based Access Control

Role-based access control (RBAC) is the way that you manage access to resources in Azure. In this lab, you will grant a user access to create and manage virtual machines in a resource group

**Grant access**

In RBAC, to grant access, you create a role assignment.
1.	In the list of Resource groups, choose the RG you have worked on
2.	Choose Access control (IAM) to see the current list of role assignments.

![rbac access](/images/RBAC-access.png)

3.	Choose Add to open the Add permissions pane.

If you don't have permissions to assign roles, you won't see the Add option.

![rbac add permissions](/images/rbac-permissions.png)

4.	In the Role drop-down list, select Virtual Machine Contributor.
5.	In the Select list, select yourself or another user.
6.	Choose Save to create the role assignment.

After a few moments, the user is assigned the Virtual Machine Contributor role at the resource group scope.

![rbac final](/images/rbac-final.png)

## 12. Lab 9 - Enable DDoS protection for your resources

*NOTE: The DDoS protection plan on the Standard Tier (Basic is Free) has a cost of ~ $3,000 a month. This means that for the use of this lab it will incur in aprox $100 which will exhaust your Azure pass credit. We recommend to use your enterprise subscription for this lab, and once you have finished revert back to DDoS protection Basic if you don’t plan to use the service anymore*

Azure automatically provides a **Basic DDoS protection** as part of the platform, at no additional charge. Always-on traffic monitoring, and real-time mitigation of common network-level attacks, provide the same defenses utilized by Microsoft’s online services. The entire scale of Azure’s global network can be used to distribute and mitigate attack traffic across regions. Protection is provided for IPv4 and IPv6 Azure public IP addresses

In this lab we will enable an **Standard DDoS protection plan**, which provides additional capabilities over the Basic service tier and are tuned specifically to Azure Virtual Network resources.  DDoS Protection Standard is simple to enable, and requires no application changes. Protection policies are tuned through dedicated traffic monitoring and machine learning algorithms. Policies are applied to public IP addresses associated to resources deployed in virtual networks, such as Azure Load Balancer, Azure Application Gateway, and Azure Service Fabric instances. Real-time telemetry is available through Azure Monitor views during an attack, and for history

As a new feature, Azure Security Center now recommends its Standard pricing tier customers to enable the Azure DDoS Protection Standard service to protect their Virtual Networks against DDoS attacks

![sec center ddos rec](/images/sec-center-ddos-rec.png)

![sec center ddos rec2](/images/sec-center-ddos-rec2.png)

![sec center ddos rec3](/images/sec-center-ddos-rec3.png)

**Create a DDoS protection plan**

A DDoS protection plan defines a set of virtual networks that have DDoS protection standard enabled, across subscriptions

1.	Select Create a resource in the upper left corner of the Azure portal.
2.	Search for DDoS. When DDos protection plan appears in the search results, select it.
3.	Select Create.
4.	Enter or select your own values, or enter,  and then select Create:

**Enable DDoS for a existing virtual network**

1.	Create a DDoS protection plan by completing the steps in Create a DDoS protection plan, if you don't have an existing DDoS protection plan.
2.	Select Create a resource in the upper left corner of the Azure portal.
3.	Enter the name of the virtual network that you want to enable DDoS Protection Standard for in the Search resources, services, and docs box at the top of the portal. When the name of the virtual network appears in the search results, select it.
4.	Select DDoS protection, under SETTINGS.
5.	Select Standard. Under DDoS protection plan, select an existing DDoS protection plan, or the plan you created in step 1, and then select Save. The plan you select can be in the same, or different subscription than the virtual network, but both subscriptions must be associated to the same Azure Active Directory tenant.

**Run a simple TCP SYN Flood attack**

In partnership with Breaking Point Cloud, we will run an ‘authorized’ DDoS attack from Breaking Point Cloud to our Public endpoint of your VNET resources. Given the fact there is no considerable traffic going through your environment, the smallest TCP SYN flood should trigger the attack and mitigation should start within minutes

Go to https://breakingpoint.cloud/

An authorize your Subscription ID as target to launch DDoS attacks

![breaking point](/images/Breaking-Point-Authorize-SubID.PNG)

Before launching the attack we confirm we have access to the public endpoint sitting on the Application Gateway we have used in the labs

![tcp site recovered](/images/tcp-syn-flood-attack-lost-web-site-recovered.PNG)

A few minutes after launching the attack, we confirm we have lost access to the endpoint

![tcp site lost](/images/tcp-syn-flood-attack-lost-web-site.PNG)

**Azure Monitor** is integrated with DDoS metrics and will see the TCP packets that have triggered an attack

![tcp syn azure monitor](/images/tcp-syn-flood-azure-monitor.PNG)

The metric **Under DDoS attack or not** is very useful

![tcp under attack or not](/images/tcp-syn-flood-under-ddos-attack-or-not.PNG)

In a few minutes mitigation should kick in place and we should be able to get access to the endpoint back again

We can see that most of the DDoS packets have been dropped by the mitigation plan

![tcp packets](/images/tcp-syn-flood-inbound-tcp-packets-ddos.PNG)

![tcp attack summary](images/tcp-syn-flood-attack-summary.PNG)



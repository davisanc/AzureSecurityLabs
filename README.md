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
- 
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









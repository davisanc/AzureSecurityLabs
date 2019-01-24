# Lab 4 – Protecting the Web Application

## Introducing Application Gateway and Web Application Firewall (WAF)

In this architecture, access to the web services running on the web VMs will be via an Application Gateway, which acts as a Layer7 HTTP reverse proxy and can load balance web traffic. The Application Gateway can be enabled with a WAF to protect our application against known vulnerabilities, such as those listed on the the [2017 OWASP Top 10 list](https://www.owasp.org/index.php/Top_10-2017_Top_10).

Internet traffic destined for the web servers should always go through a load balancer or Application Gateway where possible, with the advertised endpoint for the traffic set as a public IP address attached to the gateway.

### 4.1 - Creating the Application Gateway

We give you the option to create the AppGW using a pre-configured template or CLI commands

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

Also, you may see that the AppGateway doesn't have a public IP address after it says 'finished'. We use dynamic IP address for the AppGW and the assignment of the address will take a few minutes so bear in mind that!

NOTE: Don't do the follwing CLI commands if you just created the AppGW using the template

#### Azure CLI

This command creates the Application Gateway (for the purposes of this lab) into the **appgateway** subnet of our VNet. Please review the Azure CLI documentation for [creating an application gateway](https://docs.microsoft.com/en-us/cli/azure/network/application-gateway?view=azure-cli-latest#az-network-application-gateway-create) to see the full list of possible parameters.

```bash
az network application-gateway create --name myAppGateway --location <location> --resource-group <resource-group-name> --capacity 2 --sku Standard_Medium --http-settings-cookie-based-affinity Disabled --vnet-name ra-ntier-vnet --subnet appgateway --servers <web-server-ip-address>
```

**Please note:** the IP address of the web server required is the *internal* IP address of web server **web-vm1**. You can get the address under the **Networking** settings for the resource or by running the following CLI command:

```bash
az network nic show --name web-vm1-nic1 --resource-group <resource-group-name> --query "ipConfigurations[].privateIpAddress"
```

**Please also note** that we did not specify Public IP Address resources for this application gateway. Not specifying an IP address will create a new Public IP address, however you can be more descriptive and create these first before creating the gateway.

### 4.2 - Test the Application Gateway

Go to the AppGW resource you just created, and look for the Public IP address assigned under **Frontend public IP address**. Open a new browser tab and introduce the Public IP address, and confirm that you can reach to the Web VM IIS page through the AppGW.

### 4.3 - (Optional) - Test the Azure WAF with a DVWA

We will create a DVWA (Damn Vulnerable Web Application) VM from the marketplace, add the VM to the Application Gateway Pool and run a sequence of SQL injection and XSS attacks to test that the WAF in detection and blocking mode

**IMPORTANT:** This DVWA is managed by a 3rd party company, so Azure Pass credits cannot be used. You will need to use either an Enterprise or Pay-as-You-Go subscription.

If you are interested to test the Azure WAF or any other 3rd party WAFs using the DVWA, please report to your proctors for addional guidance and further steps to simulate some attacks like SQL injection, XSS, etc.

![DVWA](/images/dvwa-vm.PNG)

<< [Back to home page](/README.md)

<< [Previous Lab](lab-03-azure-firewall.md) . . . . . [Next Lab](lab-05-security-centre.md) >>

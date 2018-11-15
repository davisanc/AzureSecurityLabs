# Lab 7 - Extending your Data Centre to Azure in a secure way

## Using Site to Site VPN Access

**Note:** Only run this lab if you are using an Enterprise subscription or Pay-As-You-Go, **NOT** the Azure Pass. As the pfSense image is managed by a 3rd company, there is a charge for this which cannot be covered by the Azure Pass credits.

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

```bash
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

```bash
az network public-ip create --resource-group <resource-group-name> --name <ip-address-name> --sku Basic --location <location>
```

Next, we can create the Virtual Network Gateway using this IP address as a parameter:

```bash
az network vnet-gateway create --resource-group <resource-group-name> --name vpnGW-pfSense --vnet ra-ntier-vnet --gateway-type Vpn --vpn-type RouteBased --public-ip-address <vpn-ip-address-name> --sku VpnGw1 --asn 65501 --location <location>
```

Once created, you will find the BGP peer address on your VPN Gateway. This is the local address that BGP will use in your Azure VPN Gateway to initiate a BGP connection to your home gateway.

![VPN-GW-LOCAL](/images/VPN-GW-local-addr.PNG)

Now we are going to create the Local Network Gateway. Azure refers to the VPN device that sits in your onpremise network. You will need to indicate the BGP peer address, your local network behind the Firewall (or local VPN gateway) and a Private BGP ASN (I am using 65501 on the pfSense side). 

Go to your RG, click ‘Add resource’ and look for ‘ Local Network Gateway’. Use the ‘onpremise’ public ip address (example below uses 1.2.3.4)

![local nt gateway](/images/local-nt-gateway.png)

Once the local gateway is created we will define a connection to our onpremise VPN Gateway. We will use a private shared key to enable the IPSEC VPN to come up. Remember to mark BGP to ‘enabled’ on your Connection. 

**Configure the pfSense VPN Firewall**

Now, moving to the other end we will use the Web UI on the pfSense firewall to work on the Rules and VPN settings 

To configure a new tunnel, a new Phase 1 IPSEC VPN must be created. Remote Gateway will be the public IP address assigned to your Virtual Network Gateway in Azure. Leave ‘auto’ as IKE key exchange version, selecting WAN as the interface to run the VPN. For the authentication part, use the Pre-Shared Key you have defined. Use the encryption algorithm you need, in my case AES (256 bits), DH group and Hashing algorithm

![ipsec tunnels](/images/ipsec-tunnels.png)

We will then move to Phase 2. This phase is what builds the actual tunnel, sets the protocol to use, and sets the length of time to keep the tunnel up when there is no traffic. For remote network, use the VNET address space. Local subnet will be the address space on the LAN side of the pFsense

Apply changes to take them effect

![pfsense VPN2](/images/pfsense-VPN-P2.PNG)

On pfSense, go to Status, IPSEC, Overview and click **‘Connect VPN’**

*NOTE: When using BGP over a Routed IPSEC tunnel, it wouldn’t be needed the configuration and management of P2 entries. You will me managing routes in the BGP config instead of P2 entries. Routed IPSEC is a pfSense feature available in 2.4.4, my setup runs on 2.4.3 so I will create the P2 entries manually*

On the local gateway connections, as we are using BGP, **don’t forget to enable BGP or the IPSEC tunnel won’t come up!**

**Enable IPSEC traffic on the Virtual Machine pfSense NSG**

Configure a Rule to allow UDP 4500 ((IPsec NAT-T) & 500 (ISAKMP) ports 

**Enable IPSEC traffic through WAN interface of pfSense**

Configure a Rule to allow UDP 4500 ((IPsec NAT-T) & 500 (ISAKMP) ports 

![udp ports IPSEC](/images/UDP-ports-IPSEC-open.PNG)

After we added the relevant rules on the NSG and pfSense WAN interface, the connection is up and running 

![pfsense connections](/images/pfsense-connections.png)

**Enable BGP traffic through IPSEC interface**

Go to Status, System Logs, Firewall --> you can enable BGP to pass through the IPSEC interface using the logs

![pfsense BGP rule added](/images/pfSense-BGP-rule-added.PNG)

Finally, simply check that you have connectivity from the pfSense Gateway to the Web VM

![telnet port 80](/images/telnet-port-80-to-web.PNG)

<< [Back to home page](/README.md)

<< [Previous Lab](lab-06-storage-security.md) . . . . . [Next Lab](lab-08-azure-ad-rbac.md) >>
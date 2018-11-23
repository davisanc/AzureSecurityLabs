# Lab 2 - Azure Networking Logs

## Lab Overview

Network logging and monitoring in Azure is comprehensive and covers two broad categories:

- **Network Watcher**: Scenario-based network monitoring is provided with the features in Network Watcher. This service includes packet capture, next hop, IP flow verify, security group view and NSG flow logs. Scenario level monitoring provides an end to end view of network resources in contrast to individual network resource monitoring.
- **Resource monitoring**: Resource level monitoring comprises four features: diagnostics logs, metrics, troubleshooting, and resource health. All of these features are built at the network resource level.

To troubleshoot your NSG rules, enable NSG flow logs. This will enable Network watcher. 
Go to the search bar on the Portal, look for Network Watcher. Filter by your Subscription ID and Resource Group.

![network watcher](/images/Network-watcher.PNG)

**IMPORTANT! First thing**, in order for flow logging to work successfully, the Microsoft.Insights provider must be registered. If you are not sure if the Microsoft.Insights provider is registered, run the following script.

```
az provider register --namespace Microsoft.Insights
```

Then go ahead and enable NSG Flow logs on the portal. Click on the NSG you worked on on the previous lab, enable **Flow Logs**

You will need to select an storage account within your Resource Group. The NSG flow logs will be stored within a blob container of the selected storage account. More details https://docs.microsoft.com/en-us/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Alternatively, you may use the following command to enable NSG flow logs via AZ CLI or Visual Studio Code

```
az network watcher flow-log configure --resource-group resourceGroupName --enabled true --nsg nsgName --storage-account storageAccountName
```

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

<< [Back to home page](/README.md)

<< [Previous Lab](lab-01-nsg.md) . . . . . [Next Lab](lab-03-azure-firewall.md) >>

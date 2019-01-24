# Lab 9 - Enable DDoS protection for your resources

## Lab Overview

*NOTE: The DDoS protection plan on the Standard Tier (Basic is Free) has a cost of ~ $3,000 a month. This means that for the use of this lab it will incur in aprox $100 which will exhaust your Azure pass credit. We recommend to use your enterprise subscription for this lab, and once you have finished revert back to DDoS protection Basic if you don’t plan to use the service anymore*

Azure automatically provides a **Basic DDoS protection** as part of the platform, at no additional charge. Always-on traffic monitoring, and real-time mitigation of common network-level attacks, provide the same defenses utilized by Microsoft’s online services. The entire scale of Azure’s global network can be used to distribute and mitigate attack traffic across regions. Protection is provided for IPv4 and IPv6 Azure public IP addresses.

In this lab we will enable an **Standard DDoS protection plan**, which provides additional capabilities over the Basic service tier and are tuned specifically to Azure Virtual Network resources.  DDoS Protection Standard is simple to enable, and requires no application changes. Protection policies are tuned through dedicated traffic monitoring and machine learning algorithms. Policies are applied to public IP addresses associated to resources deployed in virtual networks, such as Azure Load Balancer, Azure Application Gateway, and Azure Service Fabric instances. Real-time telemetry is available through Azure Monitor views during an attack, and for history.

As a new feature, Azure Security Center now recommends its Standard pricing tier customers to enable the Azure DDoS Protection Standard service to protect their Virtual Networks against DDoS attacks.

![sec center ddos rec](/images/sec-center-ddos-rec.png)

![sec center ddos rec2](/images/sec-center-ddos-rec2.png)

![sec center ddos rec3](/images/sec-center-ddos-rec3.png)

## 9.1 - Create a DDoS protection plan

A DDoS protection plan defines a set of virtual networks that have DDoS protection standard enabled, across subscriptions

1. Select Create a resource in the upper left corner of the Azure portal.
2. Search for DDoS. When DDos protection plan appears in the search results, select it.
3. Select Create.
4. Enter or select your own values, or enter,  and then select Create:

## 9.2 - Enable DDoS for a existing virtual network

1. Create a DDoS protection plan by completing the steps in Create a DDoS protection plan, if you don't have an existing DDoS protection plan.
2. Select Create a resource in the upper left corner of the Azure portal.
3. Enter the name of the virtual network that you want to enable DDoS Protection Standard for in the Search resources, services, and docs box at the top of the portal. When the name of the virtual network appears in the search results, select it.
4. Select DDoS protection, under SETTINGS.
5. Select Standard. Under DDoS protection plan, select an existing DDoS protection plan, or the plan you created in step 1, and then select Save. The plan you select can be in the same, or different subscription than the virtual network, but both subscriptions must be associated to the same Azure Active Directory tenant.

## 9.3 - Run a simple TCP SYN Flood attack

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

![tcp attack summary](/images/tcp-syn-flood-attack-summary.PNG)

<< [Back to home page](/README.md)

<< [Previous Lab](lab-08-azure-ad-rbac.md)

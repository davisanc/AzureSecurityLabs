# Lab 5 – Azure Security Center for security recommendations

## Understand your application security posture in Azure

To take full advantage of Security Center, you need to complete the steps below to upgrade to the Standard tier and install the Microsoft Monitoring Agent

Security Center collects data from your Azure VMs and non-Azure computers to monitor for security vulnerabilities and threats. Data is collected using the Microsoft Monitoring Agent, which reads various security-related configurations and event logs from the machine and copies the data to your workspace for analysis. By default, Security Center will create a new workspace for you.
When automatic provisioning is enabled, Security Center installs the Microsoft Monitoring Agent on all supported Azure VMs and any new ones that are created. Automatic provisioning is strongly recommended.

### 5.1 - Enable your Azure subscription

1. Sign into the Azure portal.
2. Navigate to the Security Center blade
3. Click on **Start Trial** (if you have clicked on Skip, you can click on Getting Started)

![ASC-trial](/images/ASC-trial.png)

4. Click on **Install agents**, if the button has been grayed out, then it's already set to On

![ASC-agents](/images/ASC-agents.png)

5. Click on **Security policy**
6. Your subscription (Azure pass) should be listed (if it does not, close your browser session and open a new one)
7. On the line where it lists your Azure subscription (Azure pass), click on **Edit settings**
8. Set **Auto Provisioning** to **On** (if it's not already set to On)
9. Under workspace configuration, click **User another workspace** and select your Log Analytics workspace created in previous labs
10. Click on **Save**
11.	Click on **Yes** on **Would you like to reconfigure monitored VMs?**
12.	Switch back to **Security Policy** and ignore the message "Your unsaved edits will be discarded"
13.	On the line where it lists your workspace, click on **Edit settings**
14.	Click on Pricing tier, select **Standard** and click on **Save**
15.	Click on **Data collection** and select **All Events** and click on **Save**


Go to the **Security Center – Overview** which provides a unified view into the security posture of your hybrid cloud workloads, enabling you to discover and assess the security of your workloads and to identify and mitigate risk. Security Center automatically enables any of your Azure subscriptions not previously onboarded by you or another subscription user to the Free tier.

You can view and filter the list of subscriptions by clicking the Subscriptions menu item. Security Center will now begin assessing the security of these subscriptions to identify security vulnerabilities. To customize the types of assessments, you can modify the security policy. A security policy defines the desired configuration of your workloads and helps ensure compliance with company or regulatory security requirements.

Within minutes of launching Security Center the first time, you may see:

- **Recommendations** for ways to improve the security of your Azure subscriptions. Clicking the Recommendations tile will launch a prioritized list.
- An inventory of **Compute & apps, Networking, Data security, and Identity & Access** resources that are now being assessed by Security Center along with the security posture of each.

Now that you’ve upgraded to the Standard tier, you have access to additional Security Center features, including **adaptive application controls, just in time VM access, security alerts, threat intelligence, automation playbooks**, and more. Note that security alerts will only appear when Security Center detects malicious activity.

![oms global](/images/oms-global.png)

With this new insight into your Azure VMs, Security Center can provide additional recommendations related to system update status, Operating System security configurations, endpoint protection, as well as generate additional security alerts.

![oms recomm](/images/oms-recomm.png)

<< [Back to home page](/README.md)

<< [Previous Lab](lab-04-app-gateway.md) . . . . . [Next Lab](lab-06-storage-security.md) >>

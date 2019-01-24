# Lab 8 - Azure Active Directory Labs

On the next tasks we will test Azure Identity Protection, Risk Policies, MFA and Role Base Access Control

But first, we need to create a few new users in our Azure AD to have multiple identities

### Create new users in Azure AD

Create another global admin user under Azure Active Directory

Sign out from the Azure Portal, and sign in using the global admin user credentials

We will simulate how you can syncronize your on-premises Active Directory users to Azure AD. On an enterprise production environment we would use AADConnect which synchronizes the credentials to Azure AD. However, for the purpose of this lab, as we don’t have an on-premises Domain Controller, we will simply use a csv file which contains a list of credentials. We will run a set of Powershell commands to simply create users in Azure AD with the same credentials. 

Go ahead and run Powershell as administrator. You will use a users.csv file provided in the lab. This file will be sent to your lab office email account (azsecjanX@outlook.com). Sign in to office.com with your lab credentials to get the users.csv file

![get-credential](/images/get-credential.PNG)

Use the Global Admin username credentials created previously

![store-tenant](/images/store-tenant.PNG)

![build-licenses](/images/build-licenses.PNG)

![connect-msolservice](/images/connect-msolservice.PNG)

![remove-licenses](/images/remove-licenses.PNG)

![connect-azuread](/images/connect-azuread.PNG)

![import-csv](/images/import-csv.PNG)

![foreach](/images/foreach.PNG)

![new-azureadgroup](/images/new-azureadgroup.PNG)

Now, we will create a new AD group called ‘Identity Protection’ that will be used to test MFA and risky sign-in attempts

![new-azureadgroup2](/images/new-azureadgroup2.PNG)

As these users have week passwords, we will configure SSPR (Self Service Password Reset) first. 
Make sure you have ‘authentication contact info’ for your users. As we will test SSPR with the user Adam Smith, make sure you add his contact info under the following blade (you will use a mobile number that you have access to so you can receive your verification codes):

![auth-info](/images/auth-info.PNG)

You may prefer to use Powershell to add authentication info for your users. The following example would be for the user Alice Anderson. After you enter ‘Connect-MsolService’, use the global admin credentials to connect to your tenant:

![auth-infops](/images/auth-infops.PNG)

Now, we switch to the Azure Portal to configure SSPR
Go to ‘Azure Active Directory’, under ‘Password Reset’, click on ‘Self service password reset enabled’ and select the ‘Identity Protection’ group that you just created. Click on ‘Save’

![passreset](/images/passreset.PNG)

Go to ‘Authentication Methods’ and select the following, then click on ‘Save’

![passreset-auth](/images/passreset-auth.PNG)

Under ‘Registration’ select ‘No’. Click on ‘Save’
Under ‘Notifications’, select the default ‘Yes’ for ‘Notify users on passwords resets’
Now that we have enabled SSPR, lets go ahead and open a Private Browser tab and go to portal.azure.com, and click on ‘can’t access my account’

![cannotaccess](/images/cannotaccess.PNG)

Enter your Work or School account credential and you will see the next window to receive a verification code from Self service password reset:

![getback](/images/getback.PNG)

You may select send a text to your mobile phone. After you introduce the verification code you will be able to change the user password

![getback2](/images/getback2.PNG)

### Configure MFA registration policy

Navigate to Azure Active Directory > Security > MFA registration policy.
Click on ‘Get a free Premium trial to use this feature’

![mfaenable](/images/mfa.PNG)

Select ‘Enterprise Mobility and Security E5 trial’ , and click on ‘Activate’ on the following window

![e5trial](/images/e5trial.PNG)

In the next task, we will assign licenses to users that have been synced to the Office 365 portal.

1.	In a InPrivate window, navigate to https://admin.microsoft.com/AdminPortal/Home#/homepage.
  
  INFO: If needed, log in using the credentials below:
  Global Admin Username
  Global Admin Password

2.	In the middle of the homepage, click on Active users >.
3.	Check the box to select all users and click Edit product licenses.

![prodlicenses](/images/prodlicenses.PNG)

4.	On the Assign products page, click Next

![replacelic](/images/replacelic.PNG)

5.	On the Replace existing products page, turn on licenses for Enterprise Mobility + Security E5 and Office 365 Enterprise E5 and click Replace

![licenseson](/images/licenseson.PNG)

Under Azure Active Directory, Click on ‘Multi-Factor Authentication’. Select the ‘admin’ and ‘Evan Green’ user and click on ‘Enable’

![mfaenable](/images/mfaenable.PNG)

In a new InPrivate window, log in to https://portal.azure.com using the credentials of Evan Green
evang@azsecjan0outlook.onmicrosoft.com
password: <your-new-reset-password>

![addsecurity](/images/addsecurity.PNG)

### Configure Risk Policies

We are going to use Azure AD Identity Protection for the next task
Click on ‘Create a Resource’, Under Identity click on ‘Azure AD Identity Protection’, and click on ‘Create’

![AADIP](/images/AADIP.PNG)

Next, on the search bar introduce identity protection to go to Azure AD Identity Protection menu

![AADIP2](/images/AADIP2.PNG)

Let’s configure the sign-in risk policy
1.	Click on the Security blade and then select Sign-in risk policy
2.	Under Assignments users, click on Select individuals and groups and then select the all users. Click Done
3.	Under Conditions, ensure that sign-in risk is set to Medium and above
4.	Under Controls, ensure that access is set to require multi-factor authentication
5.	Set Enforce Policy to On

![signinriskpolicy](/images/signinriskpolicy.PNG)

Now, let’s configure the user-risk policy

1.	Click on the Security blade and then select User risk policy
2.	Under Assignments users, click on Select individuals and groups and then select the all users group. Click Done
3.	Under Conditions, set user risk is set to High
4.	Under Controls, ensure that access is set to require password change
5.	Set Enforce Policy to On

![userriskpolicy](/images/userriskpolicy.PNG)

Now, lets simulate a risky sign-in experience
Create a Windows 10 VM in your subscription, and download ToR browser
Using ToR browser, login to the Azure portal (portal.azure.com) and sign in with AdamS credentials. Notice how you are blocked because the user has not registered for MFA yet and is thus unable to beat the MFA challenge prompted by the risky sign-ins policy

![blocked](/images/blocked.PNG)

Now, lets sign in to the azure portal with Evan Green who has enabled MFA. Notice how you are prompted for MFA due to the risky sign-ins policy

![suspicious](/images/suspicious.PNG)

### Pull data from Microsoft Graph API

#### Get started with the API

There are four steps to accessing Identity Protection data through Microsoft Graph:

1.	Create a new app registration.
2.	Use this secret and a few other pieces of information to authenticate to Microsoft Graph, where you receive an authentication token.
3.	Use this token to make requests to the API endpoint and get Identity Protection data back.

#### Create a new app registration

1.	On the Active Directory page, in the Manage section, click App registrations.

![appreg](/images/appreg.PNG)

2.	In the menu on the top, click New application registration

3.	On the Create page, perform the following steps:

![create](/images/create.PNG)

i.	In the Name textbox, type a name for your application (e.g.: AADIP Risk Event API Application).
ii.	As Type, select Web Application And / Or Web API.
iii.	In the Sign-on URL textbox, type http://localhost.
iv.	Click Create.

4.	To open the Settings page, in the applications list, click your newly created app registration.
5.	Copy the Application ID and paste it into a new text document. This will be needed later in the lab.

#### Grant your application permission to use the API

1.	On the Settings page, click Required permissions.

![required](/images/required.PNG)

2.	On the Required permissions page, in the toolbar on the top, click Add.

3.	On the Add API access page, click Select an API.

![selectAPI](/images/selectAPI.PNG)

4.	On the Select an API page, select Microsoft Graph, and then click Select.

5.	On the Add API access page, click Select permissions.

6.	On the Enable Access page, click Read all identity risk information, and then click Select.

![enableaccess](/images/enableaccess.PNG)

7.	On the Add API access page, click Done.
8.	On the Required Permissions page, click Grant Permissions, and then click Yes.

![grantpermissions](/images/grantpermissions.PNG)

#### Get an access key

1.	On the Settings page, click Keys.

![keys](/images/keys.PNG)

2.	On the Keys page, perform the following steps:

![keyvalue](/images/keyvalue.PNG)

i.	In the Key description textbox, type a description (for example, AADIP Risk Event).
ii.	As Duration, select in 1 year.
iii.	Click Save.
iv.	Copy the key value, and then paste it into a safe location.

Since we will use this value later on, copy the Client Secret into the text file where you stored the client id.
NOTE: If you lose this key, you will have to return to this section and create a new key. Keep this key a secret: anyone who has it can access your data.

#### Authenticate to Microsoft Graph and query the Identity Risk Events API

At this point, you should have specified the following values in your text file:
•	The client ID
•	The key

#### Querying the API using PowerShell

Now that we have configured the app registration and retrieved the values needed to authenticate, we can query the IdentityRiskEvents API using PowerShell

#### See medium-risk and high-risk events

First, let’s assess how many risk events we have that are medium or high risk. These are the events that have the capability to trigger the sign-in or user-risk policies. Since they have a medium or high likelihood of user compromise, remediating these events should be a priority.
1.	Open a PowerShell ISE window and, in the script pane, type the PowerShell code below.
2.	Insert the saved Client ID and key for the values of ClientID and ClientSecret variable and click Run.

```
##Get all your medium or high-risk risk events

$ClientID       = "ClientID"        # Should be a ~36 hex character string; insert your info here
$ClientSecret   = "ClientSecret"    # Should be a ~44 character string; insert your info here
$tenantdomain   = "yourtenant.com"    # For example, contoso.onmicrosoft.com

$loginURL       = "https://login.microsoft.com"
$resource       = "https://graph.microsoft.com"
$body      = @{grant_type="client_credentials";resource=$resource;client_id=$ClientID;client_secret=$ClientSecret}
$oauth     = Invoke-RestMethod -Method Post -Uri $loginURL/$tenantdomain/oauth2/token?api-version=beta -Body $body
Write-Output $oauth
if ($oauth.access_token -ne $null) {
$headerParams = @{'Authorization'="$($oauth.token_type) $($oauth.access_token)"}
$url = "https://graph.microsoft.com/beta/identityRiskEvents?$filter=riskLevel eq 'high' or riskLevel eq 'medium'" 
Write-Output $url
$myReport = (Invoke-WebRequest -UseBasicParsing -Headers $headerParams -Uri $url)
foreach ($event in ($myReport.Content | ConvertFrom-Json).value) {
	Write-Output $event
}
} else {
Write-Host "ERROR: No Access Token"
}
```
#### Investigate a specific user

When you believe a user may have been compromised, you can better understand the state of their risk by getting all of their risk events. Similarly, if you have users that you believe may be more likely targets of compromise, you can proactively retrieve their risky events.
Since we know that Alan had some risky-sign ins, let’s query their risk events.
In the PowerShell ISE, open a new file and, in the script pane, type the PowerShell code below.
Insert the saved Client ID and key for the values of ClientID and ClientSecret variable and click Run.

```
##Get a specific user's risk events

$ClientID       = "ClientID"        # Should be a ~36 hex character string; insert your info here
$ClientSecret   = "ClientSecret"    # Should be a ~44 character string; insert your info here
$tenantdomain   = "yourtenant.com"    # For example, contoso.onmicrosoft.com

$loginURL       = "https://login.microsoft.com"
$resource       = "https://graph.microsoft.com"
$body      = @{grant_type="client_credentials";resource=$resource;client_id=$ClientID;client_secret=$ClientSecret}
$oauth     = Invoke-RestMethod -Method Post -Uri $loginURL/$tenantdomain/oauth2/token?api-version=beta -Body $body
Write-Output $oauth
if ($oauth.access_token -ne $null) {
$headerParams = @{'Authorization'="$($oauth.token_type) $($oauth.access_token)"}
$url = "https://graph.microsoft.com/beta/identityRiskEvents?`$filter=userID eq '<Alan’s user ID>'"
Write-Output $url
$myReport = (Invoke-WebRequest -UseBasicParsing -Headers $headerParams -Uri $url)
foreach ($event in ($myReport.Content | ConvertFrom-Json).value) {
	Write-Output $event
}
} else {
Write-Host "ERROR: No Access Token"
}

```


## Azure AD Role Base Access Control

Role-based access control (RBAC) is the way that you manage access to resources in Azure. In this lab, you will grant a user access to create and manage virtual machines in a resource group

### 8.1 - Grant access

In RBAC, to grant access, you create a role assignment.

1. In the list of Resource groups, choose the RG you have worked on
2. Choose Access control (IAM) to see the current list of role assignments.

![rbac access](/images/RBAC-access.png)

3. Choose Add to open the Add permissions pane.

If you don't have permissions to assign roles, you won't see the Add option.

![rbac add permissions](/images/rbac-permissions.png)

4. In the Role drop-down list, select Virtual Machine Contributor.
5. In the Select list, select yourself or another user.
6. Choose Save to create the role assignment.

After a few moments, the user is assigned the Virtual Machine Contributor role at the resource group scope.

![rbac final](/images/rbac-final.png)

<< [Back to home page](/README.md)

<< [Previous Lab](lab-07-site-to-site-vpn.md) . . . . . [Next Lab](lab-09-ddos.md) >>

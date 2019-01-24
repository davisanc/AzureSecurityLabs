# Lab 8 - Azure Active Directory Labs

## Lab Overview

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

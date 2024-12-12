# Cheatsheet

## Tools

* o365creeper\
  [https://github.com/LMGsec/o365creeper](https://github.com/LMGsec/o365creeper)
* MicroBurst\
  [https://github.com/NetSPI/MicroBurst](https://github.com/NetSPI/MicroBurst)
* AADInternals\
  [https://github.com/Gerenios/AADInternals](https://github.com/Gerenios/AADInternals)

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
// Some code
```






# Logging2Win<sup>TM</sup>
## Configuring Windows Event Logs for Security Outcomes

Windows Event Logs can provide a treasure trove of security related information. This
information can be used to aid in hunting through host-based telemetry to aid in
discovery of system/user actions that might not otherwise trigger a detection event.

This project provides a Microsoft Windows Group Policy Object \[GPO\] to make configuration
of Windows Event Logs for Security Outcomes as streamlined as possible. The GPO can be 
associated with Active Directory sites, domains, or organizational unite [OUs]. The 
provided GPO will be imported into a Domain Controller, and enabled for system policy 
enforcement.

While GPOs can be used to define registry-based policies, security options, software 
installation, manage firewalls, and more - the provided GPO will focus on Security
based event logging and can be imported/added as an additional GPO layer to an existing
policy structure. It is **not recommended, nor is it necessary**, to use this Security 
Logging GPO as the only policy object in an Active Directory structure.

## RESOURCES:
* [Group Policy Objects](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/policy/group-policy-objects)
* [Advanced Security Audit Policy Settings](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/advanced-security-audit-policy-settings)

## GETTING STARTED
Download the GPO zip and uncompress in a folder accessible to the domain controller

## INSTALLATION and DEPLOYMENT
On a Domain Controller, as an administrator, launch the Group Policy Management Center

1. Navigate to the domain for which you want to apply/enforce the Security Logging GPO settings
2. Select and right-click on Group Policy Objects under the chosen domain
3. Select "New"
4. Name the new GPO and click the OK button
5. Select the newly created GPO, and choose the "Settings" tab for the new GPO
NOTE: Your domain controller may present a dialog for Internet Explorer, as the Settings tab is seeking to render an HTML based report
6. The Settings will be empty, as we have not imported the downloaded GPO into the GPO Management Center yet
7. Right click on the newly created GPO, and select "Import Settings...". Click the button labeled "Next"
8. This is a new GPO, so you can safely ignore the "Backup" button. Click "Next"
9. Navigate to the folder where you have uncompressed the GPO zip
10. The Import Settings Wizard will show a list of any GPO backups found in that folder. Select the Security Event Logging GPO
11. You can review any changes that would be added to your GPO layers by selecting "View Settings..."
12. Continue the import by clcking "Next"
13. Scan Results should complete quickly and not have any errors related to security principals or universal naming convention paths. Click Next
14. Click Finish to complete the import process

**NOTE:**
**Enabling the imported GPO requires that it be linked with either an AD Site, Organization Unit or top-level AD Domain. The design of your Active Directory forest will determine where/how you link this new GPO.**

15. Determine where you will apply the new GPO layer, right click, and Link the Security Event Logging GPO
to that AD Object. 

**NOTE: Any AD namespace where this GPO is not linked will not provide the increased system event logging we seek for our security outcomes.**
16. Right click on the new GPO Link in your chosen OU/Domain/Site and ensure that the Security Event Logging GPO is set to Enabled.

## POLICY PUSH
By default Group Policies are refreshed every 90 minutes as a background task, and at system reboot. While the refresh interval can be
configured, we can force an update to ensure immediate effect.

If proper RPC services are available btwn the Domain Controller and client systems [RPC Services up and running, network and host-based firewalls properly configured), you can use the Group Policy Results Wizard to confirm the GPOs enabled on a particular domain client.

### The Group Policy Management Console can be used to remotely force a refresh across an entire OU.
1. Open the Group Policy Management Console on your Domain Controller
2. Right-click on the OU/Site and click the "Group Policy Update" option
3. Click "OK" to confirm and complete the Group Policy Update action

### PowerShell invoke-GPUdate cmdlet
PowerShell can be used to force a Group Policy update for both a local machine, and remote systems part of the same domain. To update the
Group Policy on a local system run the following in a PowerShell window:

Local GPUpdate using PowerShell
```
Invoke-GPUpdate -Force
```

Remote GPUpdate using PowerShell
```
$adclients = Get-ADComputer -Filter *
$adclients | ForEach-Object -Process {Invoke-GPUpdate -Computer $_.name -RandomDelayInMinutes 0 -Force}
```

**NOTE: The `-Filter` parameter can be passed a modifier to focus the client list on a specific Active Directory object, such as members of an OU/Site/etc.**

### Gpupdate via command prompt on a remote computer
The gpupdate command is a command-line utility that updates Group Policy Settings for the system on which the command is run. It does not have a remote command feature. To update the Group Policy on a local computer, run the following from a command prompt:

```
gpupdate /force
```

This command will cause the client system to check in with it's Domain Controller and update Group Policy Settings for both Computer and User policy realms defined for that system.

## CONFIRM POLICY APPLICATION
If proper RPC services are available btwn the Domain Controller and client systems [RPC Services up and running, network and host-based firewalls properly configured), you can use the Group Policy Results Wizard to confirm the GPOs enabled on a particular domain client.

### Using PowerShell to remotely check Group Policies
In a PowerShell window, enter the following code block

```
$list = Get-AdComputer -Filter 'ObjectClass -eq "Computer"' |Select -Expand Name
Foreach ($Machine in $list){
Get-GPResultantSetOfPolicy -Computer $Machine -ReportType Html -Path "C:\Users\$($Machine)_$(get-date -f MM-dd-yyyy).Html"
}
```





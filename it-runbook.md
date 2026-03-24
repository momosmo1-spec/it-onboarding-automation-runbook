# IT Pre-onboarding Runbook: New Hire Machine Setup
Version 1.0
Last Updated: February 13, 2025

## Overview
This runbook documents the standard operating procedure for setting up new hire workstations at StackFull Software. This process ensures consistent machine configuration and appropriate security measures for all new employees before their first day.

## Prerequisites
- Domain Administrator credentials
- Windows Server access
- Target workstation
- New hire information (name, department, role)

## Procedure

### 1. Domain Setup
1. Log into the workstation with local administrator credentials
   - Username: administrator
   - Password: Pa$$w0rd
2. Join the computer to the domain (contoso.com)
   - Open System Properties (Win + Break)
   - Click "Change settings" under Computer name
   - Click "Change"
   - Select "Domain" and enter: contoso.com
   - Authenticate with domain admin credentials
   - Restart the computer when prompted

### 2. User Account Creation
1. Switch to the server
2. Open Active Directory Users and Computers
3. Create new user:
   ```powershell
   New-ADUser -Name "FirstName LastName" -GivenName "FirstName" -Surname "LastName" -SamAccountName "username" -UserPrincipalName "username@contoso.com" -Enabled $true
   ```
4. Set user password:
   ```powershell
   Set-ADAccountPassword -Identity "username" -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "InitialPassword123!" -Force)
   ```

### 3. Department Group Creation
1. In Active Directory Users and Computers:
   ```powershell
   New-ADGroup -Name "DepartmentName" -GroupScope Global -GroupCategory Security
   ```
2. Add user to group:
   ```powershell
   Add-ADGroupMember -Identity "DepartmentName" -Members "username"
   ```

### 4. Department Share Configuration
1. Create department share folder:
   ```powershell
   New-Item -Path "C:\Shares\DepartmentName" -ItemType Directory
   ```
2. Set share permissions:
   ```powershell
   New-SmbShare -Name "DepartmentName" -Path "C:\Shares\DepartmentName" -FullAccess "CONTOSO\DepartmentName"
   ```
3. Create test document:
   ```powershell
   New-Item -Path "C:\Shares\DepartmentName\test.txt" -ItemType File
   ```

### 5. Organizational Unit Setup
1. Create department OU:
   ```powershell
   New-ADOrganizationalUnit -Name "DepartmentName" -Path "DC=contoso,DC=com"
   ```
2. Move objects to OU:
   ```powershell
   Move-ADObject -Identity "CN=username,CN=Users,DC=contoso,DC=com" -TargetPath "OU=DepartmentName,DC=contoso,DC=com"
   Move-ADObject -Identity "CN=DepartmentName,CN=Users,DC=contoso,DC=com" -TargetPath "OU=DepartmentName,DC=contoso,DC=com"
   ```

### 6. Group Policy Configuration
1. Create new GPO:
   - Open Group Policy Management
   - Right-click on the department OU
   - Select "Create a GPO in this domain, and Link it here"
   - Name it "DepartmentName_Policy"

2. Configure GPO settings:
   - Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options
     - Interactive logon: Message text for users attempting to log on
     - Set message: "Do not install unauthorized programs"
   
   - User Configuration > Administrative Templates > System
     - Prevent access to command prompt: Enabled
     - Prevent access to the run command: Enabled

   - User Configuration > Windows Settings > Scripts (Logon/Logoff)
     - Add logon script:
       ```batch
       net use Z: \\server\DepartmentName
       ```

### 7. Event Viewer Verification
1. On server, open Event Viewer:
   ```powershell
   Get-EventLog -LogName Security -InstanceId 4624 | Where-Object {$_.Message -like "*username*"} | Select-Object -First 1
   ```

### 8. Latest Program Installation Check
```powershell
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* | 
Select-Object DisplayName, InstallDate | 
Sort-Object InstallDate -Descending | 
Select-Object -First 1
```

### 9. Running Services Report
```powershell
$services = Get-Service | Where-Object {$_.Status -eq "Running"}
$services | Format-Table -AutoSize | Out-File C:\running_services.txt
```

## Verification Steps
- Verify user can log in with domain credentials
- Confirm department share is accessible
- Test Group Policy settings are applied
- Validate service report generation

## Troubleshooting
Common issues and solutions:
1. Domain join fails
   - Verify DNS settings point to domain controller
   - Ensure correct admin credentials
2. Share access issues
   - Check network connectivity
   - Verify group membership
   - Review share permissions

## Security Considerations
- Ensure initial password meets complexity requirements
- Review and document all permissions granted
- Verify GPO application and inheritance
- Document any deviations from standard configuration

## Contact Information
For assistance, contact:
- IT Security Manager: Damen
- Domain Administrators Group

---
End of Runbook

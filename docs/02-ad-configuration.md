# Active Directory Configuration

## Organizational Units

Created to mirror a realistic small-org structure:

- `Employees`
- `IT`
- `Sales`
- `HR`

## Users and groups

Sample users were created in their respective OUs via PowerShell:

```powershell
New-ADUser -Name "Jane Doe" -SamAccountName jdoe -UserPrincipalName jdoe@lab.internal `
  -Path "OU=Employees,DC=lab,DC=internal" `
  -AccountPassword (ConvertTo-SecureString "LabPass123!" -AsPlainText -Force) -Enabled $true

New-ADUser -Name "Mike Ross" -SamAccountName mross -UserPrincipalName mross@lab.internal `
  -Path "OU=Sales,DC=lab,DC=internal" `
  -AccountPassword (ConvertTo-SecureString "LabPass123!" -AsPlainText -Force) -Enabled $true
```

A security group was created to control share access:

```powershell
New-ADGroup -Name "Sales-Team" -GroupScope Global -Path "OU=Sales,DC=lab,DC=internal"
Add-ADGroupMember -Identity "Sales-Team" -Members mross
```

**Note:** `New-ADUser` places accounts in the built-in `Users` container by default
unless `-Path` is specified. This caused a real issue — see Troubleshooting Log.

## Domain password policy

```powershell
Set-ADDefaultDomainPasswordPolicy -Identity lab.internal `
  -MinPasswordLength 10 -PasswordHistoryCount 5 `
  -LockoutThreshold 5 -LockoutDuration 00:30:00 -LockoutObservationWindow 00:30:00
```

Verified with `Get-ADDefaultDomainPasswordPolicy`.

## File share with layered permissions

A shared folder was created on DC01, with access controlled at both the share
and NTFS level, restricted to the `Sales-Team` security group:

```powershell
New-Item -Path "C:\Shares\SalesDocs" -ItemType Directory
New-SmbShare -Name "SalesDocs" -Path "C:\Shares\SalesDocs" -FullAccess "LAB\Domain Admins"
Grant-SmbShareAccess -Name "SalesDocs" -AccountName "LAB\Sales-Team" -AccessRight Change -Force
icacls "C:\Shares\SalesDocs" /grant "LAB\Sales-Team:(OI)(CI)M"
```

Access was verified three ways:
1. `mross` (Sales-Team member) can read/write to the share
2. `jdoe` (Employees OU, not in Sales-Team) does not see the mapped drive at all
3. `jdoe` is denied even when browsing to the share by direct UNC path

This confirms the GPO drive mapping is a convenience layer, not the actual
security boundary — share and NTFS permissions are what enforce access.

## Group Policy

**Employee-Restrictions** (linked to `Employees` OU): disables Control Panel
access via `User Configuration → Administrative Templates → Control Panel →
Prohibit access to Control Panel and PC settings`.

**Sales-DriveMap** (linked to `Sales` OU): maps `\\DC01\SalesDocs` to `S:` via
`User Configuration → Preferences → Windows Settings → Drive Maps`.

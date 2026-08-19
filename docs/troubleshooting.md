# Troubleshooting

## Terraform

### `terraform validate` fails

Run:

```bash
terraform fmt -check -recursive
terraform init
terraform validate
```

If a provider schema changed, check the pinned AzureRM 4.81.0 documentation before changing resource arguments.

## AVD registration

Microsoft's current registration workflow requires the AVD Agent and Agent Boot Loader plus a valid host-pool registration key.

Check:

```powershell
Get-Service RDAgentBootLoader
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\RDInfraAgent'
```

If the host reports `INVALID_REGISTRATION_TOKEN` or `EXPIRED_MACHINE_TOKEN`, generate/rotate the registration key and rerun Terraform.

## Entra join

```powershell
dsregcmd /status
```

Confirm `AzureAdJoined : YES`.

If `AADLoginForWindows` fails, review:

```text
C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.ActiveDirectory.AADLoginForWindows
```

## Azure Files / FSLogix

Check:

```powershell
Get-SmbConnection
Get-ItemProperty 'HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\Kerberos\Parameters'
Get-ItemProperty 'HKLM:\SOFTWARE\Policies\Microsoft\AzureADAccount'
Get-ItemProperty 'HKLM:\SOFTWARE\FSLogix\Profiles'
```

Then verify the share:

```text
\\<storage-account>.file.core.windows.net\profiles
```

Port 445 must be reachable from the session host.

## Token lifetime

The repository rotates the registration key every 24 hours and sets the actual key expiry slightly before the rotation boundary. If you have a deployment window longer than that, run Terraform again to refresh the key before provisioning additional hosts.

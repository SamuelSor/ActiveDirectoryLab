# Active Directory Project with Windows Server 2019 and VirtualBox

This project sets up an Active Directory (AD) environment with 1,000 users using Windows Server 2019 and VirtualBox. The setup is designed for a network using NAT (Network Address Translation) for internet access and an internal network for local traffic.

## Prerequisites

1. **VirtualBox**: Installed on your host machine.
2. **Windows Server 2019 ISO**: Available for installation.
3. **Windows  10 ISO**: Available for installation.
4. **System Requirements**:
   - Minimum of 4 GB RAM and 100 GB disk space for the virtual machines.
   - Host machine with at least 16 GB RAM.
5. **Active Directory Planning**:
   - Domain Name: `mydomain.com`
   - Organizational Units (OUs): Predefined as needed.
   - User details: Script to generate 1,000 users.

## Network Configuration

### NAT Network
- Provides internet access to the virtual machine.
- Configured within VirtualBox under **File > Preferences > Network > NAT Networks**.

### Internal Network
- Facilitates local traffic between virtual machines.
- Configured as an **Internal Network** in VirtualBox.

## Setup Steps

### 1. Create and Configure Virtual Machine

1. Open VirtualBox and create a new VM:
   - Name: `DC`
   - Type: Microsoft Windows
   - Version: Windows Server 2019 (64-bit)
2. Assign resources:
   - Memory: 2 GB (minimum)
   - Hard Disk: 50 GB dynamically allocated.
3. Attach Windows Server 2019 ISO to the virtual DVD drive.
4. Configure network adapters:
   - Adapter 1: NAT
   - Adapter 2: Internal Network

### 2. Install Windows Server 2019
1. Boot the VM and follow the on-screen instructions to install Windows Server 2019.
2. Choose **Desktop Experience** when prompted.
3. Set an Administrator password after installation.

### 3. Configure Network Settings

#### Adapter 1 (NAT):
1. Go to **Control Panel > Network and Sharing Center > Change Adapter Settings**.
2. Set the adapter to obtain IP and DNS addresses automatically.

#### Adapter 2 (Internal):
1. Assign a static IP address (e.g., `172.16.0.1`).
2. Subnet Mask: `255.255.255.0`.

### 4. Install and Configure Active Directory
1. Open **Server Manager** and add the **Active Directory Domain Services** role.
2. Promote the server to a domain controller:
   - Select "Add a new forest" and enter `mydomain` as the domain name.
   - Complete the wizard and restart the server.

### 5. Create 1,000 Users
1. Use PowerShell to create users in bulk:

```$PASSWORD_FOR_USERS   = "Password1"
$USER_FIRST_LAST_LIST = Get-Content .\names.txt
# ------------------------------------------------------ #

$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force
New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false

foreach ($n in $USER_FIRST_LAST_LIST) {
    $first = $n.Split(" ")[0].ToLower()
    $last = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()
    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $first `
               -Surname $last `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_USERS,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
}
```

2. Customize the script as needed for specific attributes.

## Testing the Setup
1. Verify AD functionality:
   - Use **Active Directory Users and Computers** to view users and OUs.
2. Test NAT connectivity:
   - Ping external websites from the VM.
3. Test internal network:
   - Set up another VM on the internal network and ping the AD server.

## Troubleshooting

1. **Network Issues**:
   - Ensure the NAT adapter is correctly configured in VirtualBox.
   - Check firewall settings on Windows Server.
2. **User Creation**:
   - Verify PowerShell scripts for errors.
   - Ensure the Active Directory module is installed.

## Additional Resources
- [Windows Server 2019 Documentation](https://docs.microsoft.com/en-us/windows-server/)
- [VirtualBox Networking Guide](https://www.virtualbox.org/manual/ch06.html)

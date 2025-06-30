# Ansible Lab Setup

This lab demonstrates an Ansible environment with a Control Node running on WSL2 (Ubuntu) managing a Windows 11 managed node. The setup follows the Ansible architecture outlined at [tecadmin.net](https://tecadmin.net/ansible-architecture/).

## Architecture Overview

- **Control Node**: WSL2 Ubuntu with Ansible installed
- **Managed Node**: Windows 11 host configured for WinRM
- **Connection**: WinRM over HTTP for Windows management
- **Inventory**: YAML-based host definitions
- **Playbooks**: Windows-specific automation tasks

## Prerequisites

- Windows 11 with administrative privileges
- PowerShell (for initial setup)
- Internet connection for package downloads

## Setup Instructions

### 1. Enable WSL2 on Windows 11

Run the PowerShell script as administrator:

```powershell
# Run: .\01_setup_wsl.ps1
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

**Restart your computer**, then set WSL2 as default:

```powershell
wsl --set-default-version 2
```

Install Ubuntu from Microsoft Store and complete the initial setup.

### 2. Install Ansible in WSL2

Open WSL terminal and run the installation script:

```bash
# Run: ./02_install_ansible.sh
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt install python3-pip
sudo pip3 install pywinrm
sudo pip3 install pyvmomi
sudo pip3 install ansible
sudo pip3 install ansible[azure]
```

### 3. Configure Windows Host for WinRM

On the Windows 11 host, run as administrator:

```powershell
# Download and execute Ansible WinRM configuration script
$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"
(New-Object System.Net.WebClient).DownloadFile($url, $file)
powershell.exe -ExecutionPolicy ByPass -File $file
```

### 4. Create Windows User for Ansible

Create a dedicated user account:

```powershell
# Create local user
net user ansible-user AnsiblePass123! /add
net localgroup administrators ansible-user /add
```

### 5. Configure Inventory

Update the IP address in both files to match your Windows host:

- `inventory/hosts.yml` - Main inventory file
- `host_vars/win11-host.yml` - Host-specific variables

Find your Windows IP address:
```cmd
ipconfig | findstr IPv4
```

Update the `ansible_host` value in both files.

### 6. Test the Setup

Navigate to the ansible-lab directory in WSL:

```bash
cd /mnt/h/code/yl/SmartDevelop/ansible-lab
```

Test basic connectivity:
```bash
ansible windows -i inventory/hosts.yml -m win_ping
```

Run the simple hello world playbook:
```bash
ansible-playbook -i inventory/hosts.yml playbooks/simple-hello-world.yml
```

Run the advanced hello world playbook:
```bash
ansible-playbook -i inventory/hosts.yml playbooks/hello-world-windows.yml
```

## File Structure

```
ansible-lab/
├── README.md                           # This file
├── 01_setup_wsl.ps1                   # WSL2 setup script
├── 02_install_ansible.sh               # Ansible installation script
├── inventory/
│   └── hosts.yml                       # Ansible inventory
├── host_vars/
│   └── win11-host.yml                  # Host-specific variables
├── group_vars/                         # Group variables (if needed)
└── playbooks/
    ├── simple-hello-world.yml          # Basic connectivity test
    └── hello-world-windows.yml         # Advanced Windows tasks
```

## Troubleshooting

### Common Issues

1. **WinRM Connection Failed**
   - Verify Windows Firewall allows WinRM
   - Check WinRM service is running: `winrm quickconfig`
   - Verify user credentials in inventory files

2. **WSL Network Issues**
   - Find WSL IP: `ip addr show eth0`
   - Find Windows IP from WSL: `ip route show | grep default`

3. **Authentication Errors**
   - Verify user exists: `net user ansible-user`
   - Check password complexity requirements
   - Ensure user is in Administrators group

### Verification Commands

```bash
# Test Ansible installation
ansible --version

# Test inventory parsing
ansible-inventory -i inventory/hosts.yml --list

# Test Windows connectivity
ansible windows -i inventory/hosts.yml -m win_ping -v
```

## Expected Results

After successful execution:
- `C:\ansible-test.txt` created on Windows host
- `C:\ansible-hello-world.txt` created on Windows host
- Console output showing "Hello World from Ansible!"
- System information displayed in terminal

## Security Notes

- Default credentials are for lab use only
- Change passwords for production environments
- Consider using encrypted variables with Ansible Vault
- Use HTTPS/SSL for production WinRM connections
# Ansible Lab Setup

This document provides instructions on how to set up an Ansible lab environment. This setup includes an Ansible Control Node on both Windows Subsystem for Linux (WSL2) and the main Windows 11 OS, managing a Windows 11 host.

## Architecture

This lab is based on the Ansible architecture described at [tecadmin.net](https://tecadmin.net/ansible-architecture/). The key components are:

-   **Control Node**: The machine where Ansible is installed and from which you run your playbooks. In this lab, you can use either WSL2 or your main Windows 11 OS as the Control Node.
-   **Managed Node**: The machine that Ansible manages. In this lab, the managed node is a Windows 11 host.
-   **Inventory**: A file that lists the managed nodes. In this lab, the inventory is defined in `inventory/hosts.yml`.
-   **Playbooks**: YAML files that define the automation tasks to be executed on the managed nodes. Example playbooks are provided in the `playbooks/` directory.
-   **Modules**: Ansible ships with a number of modules that can be executed directly on remote hosts. For example, the `win_ping` module is used to test connectivity to a Windows host.

## Setup Instructions

### 1. Set up WSL2

Run the following commands in PowerShell as an administrator to enable WSL2:

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

After running these commands, restart your computer. Then, set WSL2 as the default version:

```powershell
wsl --set-default-version 2
```

### 2. Install Ansible in WSL2

Open the WSL terminal and run the following commands to install Ansible:

```bash
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt install python3-pip
sudo pip3 install pywinrm
sudo pip3 install ansible
```

### 3. Configure the Windows Managed Node

To allow Ansible to connect to your Windows 11 host, you need to configure WinRM. Run the following PowerShell script on your Windows 11 host as an administrator:

```powershell
# Set up WinRM for Ansible
$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"
(New-Object System.Net.WebClient).DownloadFile($url, $file)
powershell.exe -ExecutionPolicy ByPass -File $file
```

### 4. Configure the Inventory

The inventory file `inventory/hosts.yml` defines the hosts that Ansible will manage. The `host_vars/win11-host.yml` file contains the variables for the `win11-host`, including the connection details.

Update the `ansible_host` in `inventory/hosts.yml` and `host_vars/win11-host.yml` with the IP address of your Windows 11 host.

### 5. Run the Playbooks

You can run the example playbooks from your WSL2 terminal.

To test the connection to the Windows host, run the `simple-hello-world.yml` playbook:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/simple-hello-world.yml
```

To run the more advanced `hello-world-windows.yml` playbook:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/hello-world-windows.yml
```

## Testing the Setup

After running the playbooks, you can verify the following:

-   A file named `ansible-test.txt` is created in `C:\` on your Windows 11 host.
-   A file named `ansible-hello-world.txt` is created in `C:\` on your Windows 11 host.
-   The output of the playbooks in your terminal shows "Hello World from Ansible!" and other system information.
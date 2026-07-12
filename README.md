# Multi-Distro Red Team Playbook & Provisioner 

An OS-agnostic, infrastructure-as-code deployment engine designed to provision, harden, and customize Red Team operator environments via SSH. 

This repository consolidates and ports the optimization logic, package selections, and tool suites found in legacy deployment scripts into scalable, automated **Ansible YAML playbooks**.

---

##  Supported Operating Systems

The playbook dynamically detects the target host’s OS kernel and package manager, applying specialized configurations without breaking dependencies:

*   **Debian Family:** Kali Linux, Parrot OS, Ubuntu
*   **Arch Family:** BlackArch, vanilla Arch Linux

---

##  Repository Structure

```text
redteam-playbook/
├── inventory.ini             # Target infrastructure SSH details & variables
├── playbook.yml              # Central execution blueprint
└── roles/
    └── redteam_env/
        ├── tasks/
        │   ├── main.yml      # OS detection & dynamic switchboard
        │   ├── kali.yml      # Kali setup via native apt (PimpMyKali features)
        │   ├── debian.yml    # Debian/Ubuntu/Parrot setup via pipx + release binaries
        │   └── arch.yml      # Arch/BlackArch setup (BlackArch-install features)
        └── files/
            └── tmux.conf     # Standardized terminal configurations
```

---

## Integrated Core Features

*   **Workspace Standard:** Automatically enforces a standardized operation directories framework (`/root/Workspace/{Recon,Exploits,OSINT}`).
*   **Environment Enhancements:** Disables screen blanking, power-saving lockouts, and sets up custom terminal dotfiles.
*   **Automated Tool Provisioning:** Deploys a baseline red team toolkit (`ligolo-ng`, `amass`, `certipy`, `bloodyad`, `terminator`) using each platform's native mechanism — apt on Kali, `pipx` plus prebuilt release binaries on generic Debian/Ubuntu, and `pacman` on Arch — so non-Kali hosts get the tools without adding Kali repositories.
*   **Platform Tuning:** Handles Debian-specific tasks like extracting `rockyou.txt` and updates `pacman` signature keys on Arch-based targets.

---

##  Deployment Guide

### 1. Prerequisites
Ensure your local host has Ansible installed:
```bash
# Ubuntu/Debian/Kali
sudo apt update && sudo apt install ansible -y

# Arch Linux/BlackArch
sudo pacman -S ansible
```

The playbook uses modules from the `community.general` collection (`pipx` on Debian/Ubuntu, `pacman` on Arch). Install it once:
```bash
ansible-galaxy collection install community.general
```

### 2. Configure Your Inventory
Edit the `inventory.ini` file to specify your target endpoints, SSH users, and credentials (passwords or SSH keys):

```ini
[kali_nodes]
192.168.1.50 ansible_user=kali ansible_ssh_pass=kali #example

[blackarch_nodes]
192.168.1.60 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa #example

[ubuntu_nodes]
10.10.10.15 ansible_user=ubuntu #example
```

### 3. Run the Playbook
Execute the orchestration script over SSH against your infrastructure:

```bash
# Provision ALL defined infrastructure nodes simultaneously
ansible-playbook -i inventory.ini playbook.yml

# Target a specific distro group or host subset
ansible-playbook -i inventory.ini playbook.yml --limit kali_nodes

# Run tasks step-by-step to monitor execution logs
ansible-playbook -i inventory.ini playbook.yml --step
```

---


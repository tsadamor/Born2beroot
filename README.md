*This project has been created as part of the 42 curriculum by tsadamor.*

## Description
This project aims to introduce the fundamental building blocks of system administration through setting up a virtual machine. The goal is to set up a secure Debian server without a graphical user interface, implementing strict partition setups via LVM, enhanced security frameworks, rigorous user/sudo management, firewall constraints, and an automated monitoring system.

### 1. Conceptual Overview
* **How a Virtual Machine works:** A VM is a software-defined emulation of a physical computer. It runs an operating system (Guest) on an isolated layer abstracted from the physical hardware (Host) via a Hypervisor.
* **Purpose of Virtual Machines:** VMs allow for efficient resource utilization, sandbox testing environments, easy cloning/snapshot rollbacks, and running multiple isolated operating systems on a single physical machine.
* **How LVM works:** Logical Volume Manager abstracts physical storage devices. It pools physical disks into Volume Groups (VG), which are then sliced into Logical Volumes (LV). This allows dynamic resizing of partitions without re-partitioning the raw disk.

### 2. Technical Comparisons

| Comparison Category | Option A (Chosen) | Option B |
| :--- | :--- | :--- |
| **Operating System** | **Debian:** Rock-solid stability, predictable release cycles, large community, lightweight minimal install. Ideal for learning fundamentals. | **Rocky Linux:** RHEL-downstream, enterprise-focused, utilizes SELinux and firewalld by default. Complex setup. |
| **Package Manager** | **apt:** High-level package management tool, user-friendly output, standard for Debian-based systems. | **aptitude:** More advanced, includes a text-based user interface (TUI), better at resolving complex dependency conflicts. |
| **Security Framework** | **AppArmor:** Path-based Access Control. Easier to configure, uses profiles bound to specific program paths to restrict system capabilities. | **SELinux:** Label-based Mandatory Access Control (MAC). Highly granular but extremely complex, assigning security contexts to every file and process. |
| **Firewall** | **UFW:** Uncomplicated Firewall. A user-friendly front-end for iptables/nftables, straightforward syntax for standard rule-making. | **firewalld:** Zone-based dynamic firewall daemon, supports advanced network environments and stateful rules. |

---

### 3. Design Choices & Security Policies
* **Partitioning:** Configured encrypted partitions utilizing LVM to safeguard data at rest.　Established a clean, minimal layout featuring a dedicated root (`/`) volume and a dynamic `[SWAP]` allocation inside the encrypted physical volume (`sda5_crypt`), fully satisfying the project's strict mandatory security architecture specifications.
* **Password Policy Advantages:** Enforcing minimum length, character diversity, and aging parameters drastically decreases the surface for brute-force attacks and credential stuffing.
* **Password Policy Disadvantages:** Strict rotations can lead to "password fatigue," causing users to write down complex strings or use predictable incrementing patterns.
* **Sudo Constraints:** Restricted command execution vectors (`secure_path`), enabled compulsory TTY execution, capped incorrect attempts to 3, and configured centralized output/input logging in `/var/log/sudo/`.

---

## Instructions

### 1. Simple Setup & Service Verification
* **Check that no Graphical Interface is running:**
  Ensure the VM boots directly into the Command Line Interface (CLI). No `X.org`, `Wayland`, or desktop managers should be active.
* **Check OS Version:**
  ```bash
  head -n 2 /etc/os-release
  ```
* **Check AppArmor Status:**
  ```bash
  sudo systemctl status apparmor
  ```
* **Check UFW Status:**
  ```bash
  sudo systemctl status ufw
  ```
* **Check SSH Service Status:**
  ```bash
  sudo systemctl status ssh
  ```

### 2. Password Policy & User Management
* **Check Password Complexity Policy (PAM):**
  ```bash
  sudo cat /etc/pam.d/common-password
  ```

* **Check Password Aging Policy:**
  ```bash
  sudo cat /etc/login.defs
  ```

* **Verify Aging Policy Application on Users:**
  ```bash
  sudo chage -l tsadamor
  ```
* **Verify your login user exists and check its groups**
  ```bash
  id user
  ```
* **Create a new user**
  ```bash
  sudo adduser new_user
  ```
* **Create a new group named**
  ```bash
  sudo addgroup new_group
  ```
* **Assign a user to a group:**
  ```bash
  sudo usermod -aG new_group new_user
  ```
* **Verify the new user's groups:**
  ```bash
  id new_user
  ```

### 3. Hostname & Partitions Check
* **Check the current Hostname**
  ```bash
  hostname
  ```
* **Modify the Hostname to an assigned name**
  ```bash
  sudo hostnamectl set-hostname <assigned_name>
  ```
* **Reboot the machine to apply the change and verify it:**
  ```bash
  sudo reboot
  ```
* **Restore the Hostname back to the original format after validation:**
  ```bash
  sudo hostnamectl set-hostname old_hosname
  sudo reboot
  ```
* **View the LVM Partition structure and compare size configurations:**
  ```bash
  lsblk
  ```

### 4. Sudo Policy & Log Auditing
* **Verify Sudo is installed properly:**
  ```bash
  sudo -V
  ```
* **Show the strict sudo rule file implementation:**
  ```bash
  sudo visudo
  # Or alternatively check via visudo
  sudo cat /etc/sudoers.d/sudo_config
  ```
* **Verify that the sudo log directory exists and contains active log entries:**
  ```bash
  ls -l /var/log/sudo/
  ```
* **Run a random command using sudo to test log updates:**
  ```bash
  sudo apt update
  ```
* **Verify that the history and outputs inside `/var/log/sudo/` have dynamically updated:**
  ```bash
  sudo cat /var/log/sudo/sudo.log
  ```

### 5. UFW Firewall Configurations
* **List the active UFW rules**
  ```bash
  sudo ufw status numbered
  ```
* **Add a new rule to open a port**
  ```bash
  sudo ufw allow 1000/tcp
  ```
* **Verify the port has been added successfully to the active rules:**
  ```bash
  sudo ufw status
  ```
* **Delete the newly created rule**
  ```bash
  sudo ufw delete allow 1000/tcp
  ```
* **Verify the firewall is back to its original state:**
  ```bash
  sudo ufw status
  ```

### 6. SSH Constraints
* **Check which SSH configuration points to**
  ```bash
  sudo grep -i "Port" /etc/ssh/sshd_config
  ```
* **Check Root Login via SSH**
  ```bash
  sudo grep -i "PermitRootLogin" /etc/ssh/sshd_config
  ```
* **Test logging in via SSH using the newly created user from your Host machine terminal:**
  ```bash
  ssh new_user@127.0.0.1 -p 4242
  ```

### 7. Script Monitoring & Cron Schedule
* **Display the shell script implementation logic:**
  ```bash
  sudo cat /usr/local/bin/monitoring.sh
  ```
* **View root's crontab schedule:**
  ```bash
  sudo crontab -l
  ```
* **Change the script execution interval to 1 minute**
  ```bash
  sudo crontab -e
  # Change line from: */10 * * * * /usr/local/bin/monitoring.sh
  # Change line to:   */1 * * * * /usr/local/bin/monitoring.sh
  ```
* **Restore cron runtime back to 10 minutes after validation:**
  ```bash
  sudo crontab -e
  # Revert back to: */10 * * * * /usr/local/bin/monitoring.sh
  ```

---

## Resources
* **Official Documentation Referenced:** Debian GNU/Linux Installation Guide, Linux man page.
* **AI Usage:** Documentaion Support, Conceptual Learning

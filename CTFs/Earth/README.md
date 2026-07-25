# Earth

> A technical walkthrough of the **Earth** machine from VulnHub, documenting the methodology used to obtain initial access in a controlled lab environment.

---

## Overview

**Earth** is an intentionally vulnerable Linux virtual machine from **VulnHub**. This write-up documents my approach to identifying the target, enumerating exposed services, discovering hidden resources, recovering credentials, and obtaining initial access.

The privilege escalation phase is currently in progress and will be documented upon successful completion.

---

## Machine Information

| Property         | Value                 |
| ---------------- | --------------------- |
| Platform         | VulnHub               |
| Machine          | Earth                 |
| Operating System | Linux                 |
| Difficulty       | Beginner/Intermediate |
| Environment      | Oracle VirtualBox     |
| Objective        | Obtain Root Access    |
| Status           | In Progress           |

---

## Lab Environment

**Attacker Machine**

* Kali Linux

**Target Machine**

* Earth (VulnHub)

**Network**

* Bridged Adapter (VirtualBox)

---

## Tools Used

* arp-scan
* Nmap
* DIRB
* CyberChef
* Firefox
* OpenSSH (planned)
* Linux CLI Utilities

---

## Reconnaissance

The target machine was first identified by performing an ARP scan across the local network.

```bash
arp-scan --localnet
```

The VirtualBox guest was identified by its Oracle-assigned MAC address prefix (`08`), allowing the target IP address to be distinguished from other devices on the network.

After identifying the target, an Nmap scan was performed to enumerate exposed services.

```bash
nmap -APn -p- <TARGET-IP>
```

The scan revealed three primary services:

* SSH
* HTTP
* HTTPS

The SSL certificate information also revealed additional hostnames:

* earth.local
* terratest.earth.local

Since accessing the application directly via its IP address did not return the expected content, the discovered hostnames were added to the local hosts file.

```text
/etc/hosts
```

This allowed both virtual hosts to be accessed successfully through the browser.

---

## Enumeration

Directory enumeration was performed against both discovered virtual hosts.

```bash
dirb http://earth.local
```

```bash
dirb https://terratest.earth.local
```

The enumeration revealed several interesting resources, including a `robots.txt` file on the HTTPS virtual host.

Inspection of the `robots.txt` file revealed multiple wildcard entries and a reference to a file named:

```text
testingnotes.txt
```

Accessing this file exposed valuable information regarding the application's implementation, including:

* The use of XOR encryption
* A reference to another file named `testdata.txt`
* A potential username:

```text
terra
```

The referenced `testdata.txt` file appeared to contain the encryption key used during application testing.

---

## Credential Recovery

While examining the homepage of **earth.local**, several long hexadecimal strings were observed.

These values were analyzed using **CyberChef** by converting the hexadecimal data and applying XOR decryption with the key obtained from `testdata.txt`.

One of the decoded outputs produced readable plaintext resembling:

```text
Earthclimatechangebad4humans
```

The repeated plaintext appeared to represent a potential password and was selected for authentication testing.

---

## Initial Access

Earlier directory enumeration had revealed an administrative login page.

Using the recovered credentials:

**Username**

```text
terra
```

**Password**

```text
Earthclimatechangebad4humans
```

authentication was successful.

The application presented an administrative command interface capable of executing commands.

Basic enumeration was performed using:

```bash
locate 'flag'
```

The command revealed the location of `flag.txt`, which was successfully accessed, confirming the initial foothold on the target system.

---

## Privilege Escalation

Privilege escalation is currently in progress.

The next objective is to enumerate the target further, identify privilege escalation vectors, and obtain root access.

This section will be updated upon completion of the assessment.

---

## Lessons Learned

* ARP scanning is an effective method for identifying live hosts within a local Layer 2 network.
* MAC address vendor prefixes can help distinguish virtual machines from physical hosts.
* SSL certificate information may reveal additional virtual hostnames.
* Directory enumeration can expose sensitive files not linked from the main application.
* Information disclosure files often provide valuable clues regarding usernames, encryption methods, and application functionality.
* CyberChef is an effective tool for decoding and analyzing encoded application data.
* Correlating information gathered from multiple enumeration techniques can lead to successful credential recovery.

---

## References

* Platform: VulnHub
* Machine: Earth
* CyberChef
* Nmap
* DIRB

---

## Disclaimer

This assessment was performed in a controlled lab environment using an intentionally vulnerable virtual machine. The information provided is intended solely for educational purposes.

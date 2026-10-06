# Network Security and Threat Analysis

## Purpose

This area focuses on finding and understanding malicious activity on a network. The labs cover the core tasks of a security analyst: analysing packet captures to reconstruct an attack, showing how unencrypted traffic exposes credentials, identifying malware from a file hash, and tracing an open port back to the process behind it. Each investigation ends with a verdict and a practical response, such as blocking an IP, removing a malicious file or closing unnecessary ports.

## Labs Completed

| Project | Lab | Objective | Tools used | Lessons learnt |
|---|---|---|---|---|
| [WebStrike: Packet Capture Analysis](https://github.com/Osita-Odo/Network-Security-and-Threat-Analysis/blob/main/Wireshark-Webstrike-Lab.md)<br>*Platform: CyberDefenders* | [WebStrike PCAP Investigation] | Reconstruct a web attack: attacker origin, User-Agent, web shell, upload directory and exfiltrated file. | Wireshark (Endpoints, display filters, Follow HTTP/TCP Stream), ipgeolocation.io | Double extensions such as `image.jpg.php` can bypass weak upload checks. |
| [Vulnerability Assessment](https://github.com/Osita-Odo/Vulnerabilty-Assessment-Wirshark-Nmap-Task-Manager-Powershell)<br>*Platform: Kali Linux on Oracle VirtualBox, Windows host* | [Packet Sniffing] | Capture login credentials sent over HTTP. | Wireshark, Firefox | Anyone on the network path can read HTTP credentials, so HTTPS is essential. |
| | [Malware Identification via Hash Analysis] | Identify a file from its SHA-256 hash and attribute it to a malware family. | VirusTotal | Detection ratio, filename aliases and family attribution together build a verdict. |
| | [Port Investigation] | Trace an open port to the process and service behind it. | Nmap, PowerShell (`netstat -ano`), Task Manager | Check which host a command is querying; Linux `netstat` could not see the Windows machine. |

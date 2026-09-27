# Practical Vulnerability Assessment: Packet Sniffing, Malware Identification & Port Investigation

## Objective

This project is a practical demonstration of offensive and defensive security techniques across three investigations. The first captures plaintext login credentials over the insecure HTTP protocol using packet sniffing, showing why unencrypted traffic is a vulnerability. The second analyzes a suspicious file hash to identify malware and attribute it to a known family. The third performs a port scan against a Windows host and traces an open port back to the exact process and service listening on it, verifying whether it is legitimate. Together, the investigations combine traffic analysis, malware identification, port scanning, and service verification, demonstrating hands-on use of Wireshark, Nmap, VirusTotal, Windows PowerShell, and Windows Task Manager, along with core network troubleshooting commands.

## Skills Learned

* Capturing and analyzing live network traffic in Wireshark, filtering by protocol and isolating POST and GET requests.
* Recognizing the security weakness of HTTP (port 80): credentials sent in plaintext can be recovered from captured packets.
* Analyzing file hashes with VirusTotal, interpreting community detection ratios, and attributing a sample to a malware family.
* Recognizing SHA-256 hashes and how a single sample can appear under multiple filenames and file types.
* Port scanning with Nmap to enumerate open ports on a host.
* Tracing an open port to its owning process using `netstat -ano` (PID lookup) and Windows Task Manager, then verifying the service.
* Troubleshooting when Linux socket-listing commands (`netstat`, `ss`) return no data by pivoting to the correct host's native tooling.
* Distinguishing legitimate system services from suspicious activity and understanding the risk posed by unnecessary open ports.

## Tools Used

* Kali Linux running in Oracle VirtualBox.
* Wireshark for packet capture and traffic analysis.
* Nmap for port scanning.
* VirusTotal for file hash reputation and malware family attribution.
* Windows PowerShell (`netstat -ano`) and Windows Task Manager for process and service investigation.
* Network troubleshooting commands: `netstat -tulpn`, `ss -tulpn`.

---

## Investigation 1: Network Vulnerability Scanning: Packet Sniffing

**Aim:** To monitor data packets and capture login credentials, taking into account the different protocols and action requests such as POST and GET. Analysis was performed on the host with IP `10.0.2.15` communicating over HTTP.

### Step 1: Launch Wireshark

*Ref 1: Opening Wireshark from the Kali Linux application menu*

<img width="600" height="300" alt="Launching Wireshark from the Kali application menu" src="https://github.com/user-attachments/assets/266b547f-0b84-466f-b40e-c366b0107e6e" />

Selected the application icon at the top left of Kali Linux (marked in green) and used the search box to find and open Wireshark.

### Step 2: Select the capture interface

*Ref 2: Selecting the eth0 interface for capture*

<img width="600" height="300" alt="Wireshark interface selection showing eth0" src="https://github.com/user-attachments/assets/f905e8f9-6372-4574-95b7-418336772d16" />

With Wireshark open, selected the `eth0` interface to begin capturing traffic (working from tab 2).

### Step 3: Browse to the target login page

*Ref 3: Opening the practice website in Firefox*

<img width="600" height="300" alt="Opening www.techpanda.org in Firefox on tab 1" src="https://github.com/user-attachments/assets/181a3cd9-3d59-495a-9c6b-8e5af962c72e" />

Switched to tab 1 and opened Firefox, browsing to the practice website `www.techpanda.org` (used purely for this simulation).

### Step 4: Submit login credentials while capturing

*Ref 4: Entering test credentials on the login page*

<img width="600" height="300" alt="Login page with test credentials entered" src="https://github.com/user-attachments/assets/54cb8307-30e6-4f86-a5a9-fcfec6b70796" />

Entered a deliberately incorrect email and password while the capture ran, generating a login request. Browser actions are recorded under the Info column of the captured packets; the focus here was on the POST and GET requests.

* Email: `gshddhhd@gmail.com`
* Password: `thththth`

### Step 5: Recover the credentials from the capture

*Ref 5: HTTP POST request revealing the submitted credentials in plaintext*

<img width="600" height="300" alt="Wireshark showing the HTTP POST request with plaintext login details" src="https://github.com/user-attachments/assets/9de8f16d-cb28-4c9c-a5ef-e75c3fc53902" />

Returning to Wireshark, traffic had already been captured. Filtering on the HTTP protocol for IP `10.0.2.15` and locating the POST action under the Info column exposed the login details in plaintext. Because HTTP operates over port 80 without encryption, the credentials submitted in the login request were fully readable, demonstrating the core vulnerability of unsecured web traffic.

---

## Investigation 2: Malware Identification via Hash Analysis

**Hash analyzed:** `f02ce710874cd773c3c32730dc2536d14f4e419bf828d4a27664a24af5d4484f`
**Tool used:** VirusTotal
**Hash type:** SHA-256

*Ref 6: VirusTotal analysis of the file hash*

<img width="600" height="300" alt="VirusTotal detection results for the analyzed SHA-256 hash" src="https://github.com/user-attachments/assets/76b4b18a-ea16-4886-aad3-1fc7aeed728b" />

**Assessment: the file is a malicious trojan.** Reasoning:

* The community detection ratio was **42/64**, indicating the majority of security vendors flagged it as malicious.
* The same hash appeared under multiple filenames and file types, a common evasion trait:
  1. `f02ce710...5d4484f.elf`
  2. `f02ce710...5d4484f.elf` (variant)
  3. `jmlvg.exe`
  4. `Polar.ppc.elf`

**Family:** the sample belongs to the **Condi** malware family, a botnet that targets vulnerable network devices running Linux.

---

## Investigation 3: Nmap Scan, PowerShell & Task Manager Correlation

**Purpose:** To find out which ports are open on a Windows host, why they are open, and which service is listening on them.

**Tools:** Nmap (Kali Linux), Windows PowerShell, Windows Task Manager.

### Step 1: Scan for open ports

*Ref 7: Nmap scan revealing open ports*

<img width="600" height="300" alt="Nmap scan output showing open ports including 135/tcp" src="https://github.com/user-attachments/assets/629dc4ad-c80a-4351-b279-6060d84205c9" />

```bash
nmap 192.168.20.10   # IP randomized for this report
```

The scan returned several open ports. Port `135/tcp` was chosen as the case to investigate further.

*(The actual scanned IP is withheld for security reasons.)*

### Step 2: Attempt to identify the listening service on Linux

Tried to list listening services from Kali:

```bash
sudo netstat -tulpn
sudo ss -tulpn
netstat -tulpn
```

The columns displayed but no data populated, since the scanned host was a Windows PC. This prompted a pivot to Windows-native tooling.

### Step 3: List active connections in PowerShell

*Ref 8: netstat -ano in PowerShell mapping port 135 to a PID*

<img width="600" height="300" alt="PowerShell netstat -ano output showing PID 1592 for port 135" src="https://github.com/user-attachments/assets/5f6f8537-06b6-4804-a79b-e96918f8aeb7" />

Running PowerShell as administrator:

```powershell
netstat -ano
```

The output showed that port `135/tcp` mapped to **PID 1592**. The process ID (PID) is what links a connection to a specific running service.

### Step 4: Identify the process behind the PID

*Ref 9: Windows Task Manager resolving PID 1592 to a process*

<img width="600" height="300" alt="Task Manager Details tab showing PID 1592 as svchost.exe" src="https://github.com/user-attachments/assets/49b39c35-9658-487a-9bbe-f28e66c2917d" />

Opened Windows Task Manager, selected the **Details** tab to view running processes with their PIDs, and used `Ctrl+F` to search for PID 1592. The result identified the process as `svchost.exe`.

**Verification:** research confirmed `svchost.exe` (hosting the `msrpc` service on port 135) is a legitimate Windows service.

**Implication:** open ports can serve as entry points for attackers. Although `svchost.exe` is legitimate, other unnecessary ports should be closed to reduce the attack surface and avoid exploitation.

---

## Conclusion

* HTTP's lack of encryption makes credential theft trivial for anyone on the network path; this is the practical case for HTTPS/TLS everywhere.
* A hash plus VirusTotal is often enough to triage a suspicious file: the detection ratio, multiple aliases, and family attribution together build the verdict.
* Enumerating an open port is only the first step; the real analyst work is tracing it to a process (PID) and a service, then judging whether it is legitimate.
* Knowing when a tool is the wrong fit (Linux `netstat` against a Windows host) and pivoting to the right one is itself a core troubleshooting skill.

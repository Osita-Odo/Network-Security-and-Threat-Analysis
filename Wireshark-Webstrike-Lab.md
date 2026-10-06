# WebStrike: Packet Capture Analysis Report

A blue-team network forensics investigation based on the CyberDefenders "WebStrike" challenge.

## Objective

This report presents a network forensics investigation of a web-based attack. The goal was to analyze a packet capture (`WebStrike.pcap`) in Wireshark, trace the attack, identify the attacker and their location, and determine what was compromised. The investigation reconstructs the full intrusion: geolocating the attacker, recovering their User-Agent, identifying the malicious web shell that was uploaded through a vulnerable file-upload feature, locating the upload directory, and confirming the attempted exfiltration of a sensitive system file. Each finding is tied back to a concrete incident-response action such as IP geo-blocking, filtering rules, and file remediation.

**Tools used:** Wireshark and an online IP geolocation service (ipgeolocation.io).
**Evidence file:** `WebStrike.pcap` provided for the challenge (355 packets, two IPv4 endpoints).

## Skills Learned

* Packet capture (PCAP) analysis in Wireshark.
* Identifying hosts and traffic volume using the Statistics menu and Endpoints submenu.
* IP geolocation and threat intelligence using ipgeolocation.io.
* Recovering the attacker's User-Agent to support the creation of filtering rules.
* Identifying web-shell upload and data-exfiltration behaviour.
* Matching network evidence (Indicators of Compromise) to incident-response actions such as IP geo-blocking, filtering rules, and file remediation.

## Tools Used

* Wireshark for packet capture analysis, following HTTP and TCP streams, and applying display filters.
* ipgeolocation.io for attacker IP geolocation and threat intelligence.
* Datasets: `WebStrike.pcap` (355 packets between two IPv4 endpoints).

---

## Initial Triage: Mapping the Conversation

Before answering the questions, I opened the capture and reviewed the overall traffic using Wireshark's Statistics menu. I selected the Statistics menu and the Endpoints submenu to see the IP addresses of the hosts interacting in the captured network.

The Endpoints window (IPv4 tab) shows two hosts: `24.49.63.79` (web server) and `117.11.88.124` (attacker), each exchanging 355 packets. The external address `117.11.88.124` is the IP to investigate.

<img width="809" height="446" alt="image" src="https://github.com/user-attachments/assets/b5133329-2a2b-4fb6-847a-7bedff48395f" />



*Ref 1: Endpoints window (IPv4 tab) identifying the web server and attacker*

---

## Task 1: Geographical Origin of the Attack

This task aimed at identifying the geographical origin of the attack, which helps in implementing IP geo-blocking measures and analyzing threat intelligence. I took the attacker IP `117.11.88.124` and ran it through the ipgeolocation.io website.

The lookup resolved the IP to **Tianjin, China**, with hostname `dns124.online.tj.cn`, District Nankai, and State code `CN-TJ`.

<img width="975" height="530" alt="image" src="https://github.com/user-attachments/assets/09bd1603-45de-4a19-9733-0c600dff0488" />


*Ref 2: IP geolocation lookup for 117.11.88.124*

---

## Task 2: Attacker's User-Agent

Knowing the attacker's User-Agent assists in creating filtering rules. To find it, I:

1. Right-clicked a packet from the attacker.
2. Selected **Follow**.
3. Selected **HTTP Stream** to reconstruct the HTTP conversation and read the request headers.

The output shows the User-Agent is:

```
Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
```

This indicates the attacker was operating from a Linux host and running Mozilla Firefox 115.

<img width="974" height="394" alt="image" src="https://github.com/user-attachments/assets/9e46ddf5-da8c-4abd-8fd8-f8eacaec7c7b" />


*Ref 3: Follow HTTP Stream revealing the attacker's User-Agent*

---

## Task 3: Malicious Web Shell That Was Uploaded

This task required determining whether any vulnerabilities were exploited and the name of the malicious web shell that was successfully uploaded. I applied the display filter `http.request.method == POST` to find requests using the POST method, then right-clicked the captures to follow the HTTP streams and trace the POST requests.

<img width="975" height="441" alt="image" src="https://github.com/user-attachments/assets/598b4aa0-e60c-4e15-87f0-fbd1d732fdf9" />

*Ref 4: Filtering for POST requests*

On the HTTP stream, I confirmed the file was successfully uploaded to the server. The uploaded file is `image.jpg.php`. This file bypassed the security check and was successfully uploaded because `.jpg` was appended to the filename, making it difficult to detect (a double-extension bypass).

<img width="967" height="438" alt="image" src="https://github.com/user-attachments/assets/7ec4ad50-3d5d-43af-9b35-f643512cc634" />


*Ref 5: HTTP stream confirming the image.jpg.php web shell upload*

---

## Task 4: Directory Used to Store Uploaded Files

The aim was to identify the directory where the uploaded `image.jpg.php` file is stored, which is important for locating the vulnerable page and removing any malicious files. I reused the POST filter to find the request with the POST method.

<img width="975" height="441" alt="image" src="https://github.com/user-attachments/assets/ecdad25b-593c-4070-9cc0-fbcc771d92ee" />


*Ref 6: Re-applying the POST filter*

Then I filtered for the packet with the uploaded file using:

```
http.request.uri contains "image.jpg.php"
```

<img width="975" height="429" alt="image" src="https://github.com/user-attachments/assets/355b4bd0-9c54-4592-a4d2-7b119973e852" />


*Ref 7: Filtering for the uploaded file by URI*

Right-clicking, selecting **Follow**, and then **HTTP Stream** reveals the directory the application writes uploads into: `/reviews/uploads/`. This is the location that must be cleaned up during remediation.

<img width="975" height="514" alt="image" src="https://github.com/user-attachments/assets/2616b075-15ca-4151-9b2d-469e56f2215d" />



*Ref 8: HTTP stream revealing the upload directory /reviews/uploads/*

---

## Task 5: Port Targeted for Unauthorized Outbound Communication

This task required finding the port targeted by the malicious web shell for establishing unauthorized outbound communication. I filtered for the POST request method and followed the HTTP stream, which revealed that the targeted port is **8080**.

<img width="975" height="444" alt="image" src="https://github.com/user-attachments/assets/53837481-7e6d-4c20-8298-9202b306e547" />

*Ref 9: HTTP stream revealing the targeted port 8080*

---

## Task 6: File the Attacker Attempted to Exfiltrate

Identifying the importance of the compromised data helps in prioritization during incident response. I first ran the filter:

```
(tcp.port==8080) && (ip.src==24.49.63.79)
```

Since the attacker targeted port 8080 of the server, I right-clicked the first highlighted packet on port 8080 to select **Follow** and then **TCP Stream**.

<img width="934" height="362" alt="image" src="https://github.com/user-attachments/assets/6eb76a75-dcb1-44f3-9caf-f66589565df3" />


*Ref 10: Filtering port 8080 traffic and following the TCP stream*

Using the TCP stream, I found that the file is `/etc/passwd`, which is an account database. The `curl -X POST -d /etc/passwd` command reads the file and posts it to the attacker's listener at port 443, confirming an attempt to exfiltrate system user information.

<img width="971" height="437" alt="image" src="https://github.com/user-attachments/assets/00936a60-a57b-4893-a179-7f8c50408741" />

*Ref 11: TCP stream confirming the /etc/passwd exfiltration attempt*

---

## Analysis Summary

| # | Question | Finding |
|---|----------|---------|
| 1 | Origin city | Tianjin, China (AS4837 China Unicom) |
| 2 | User-Agent | `Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0` |
| 3 | Web shell | `image.jpg.php` |
| 4 | Upload directory | `/reviews/uploads/` |
| 5 | Outbound port | 443 |
| 6 | Exfiltrated file | `/etc/passwd` |

In summary, an attacker in Tianjin, China with IP `117.11.88.124` reached the shoporoma.com web server (`24.49.63.79`) and uploaded a malicious web shell disguised with a `.jpg` extension through a vulnerable upload feature. The attacker then used it to exfiltrate `/etc/passwd` to a listener on port 443, with the targeted server port being 8080.

---

## Recommendations

Based on the Indicators of Compromise recovered above:

1. **Geo-block and filter the attacker IP** `117.11.88.124` at the firewall, informed by the Tianjin / China Unicom attribution.
2. **Remediate the web shell**: remove `image.jpg.php` from `/reviews/uploads/` and audit the directory for any other uploaded files.
3. **Fix the upload vulnerability**: enforce server-side validation of file type and extension so double-extension files like `image.jpg.php` cannot bypass checks; store uploads outside the web root.
4. **Create detection rules** from the recovered User-Agent and the outbound connection pattern to flag similar activity.
5. **Restrict egress traffic** so the web server cannot make arbitrary outbound connections on ports such as 8080 and 443.

---

# Email Threat Investigation with Wireshark PCAP Analysis

Investigated four packet captures (A.pcap to D.pcap) to identify a malicious sender, find phishing and extortion emails, and extract attachments from SMTP traffic using Wireshark and command-line email tools.

Context: Completed as Project 1 of CodePath CYB102 (Cybersecurity Analysis), Summer 2025.

Tools: Wireshark, munpack, tnef, .eml file inspection

# Overview

Scenario: Four network captures contained email (SMTP) traffic. The goal was to work out which host was the "bad apple" sending suspicious messages, and to identify the phishing emails hidden in the captures.

My role: Sole analyst. I filtered each capture, exported suspicious emails, extracted their contents, and correlated findings across the files.

Result: Identified 192.168.1.4 as the malicious sender and found three distinct extortion-style phishing emails in C.pcap.

# Summary of Findings


| Capture | What I found |
| :--- | :--- |
| **A.pcap** | Email with subject "Testing testing 1 2 3" carrying multiple attachments, including an image and a patch file with C code. Sent from 192.168.1.4. |
| **B.pcap** | A single internal SMTP transmission from 10.10.1.4 with a .pcap file attached. Also an attempted connection to 192.168.1.1, which was unreachable. |
| **C.pcap** | The phishing emails: no attachments, but each quotes a password and threatens the recipient, demanding payment in Bitcoin. Also showed disconnect errors referencing pool.washdc.fios.verizon.net. |
| **D.pcap** | Email with subject "Test message for capture" with a words.txt file and a C source file attached. Sent from 192.168.1.4. |

# Malicious actor: 192.168.1.4 
It is the source of the emails with attachments in A and D

# Methodology
i. Isolate email traffic. Filtered each capture with smtp to list all mail transmissions.

ii. Find emails with content. Used smtp contains DATA to find messages that carried a body or attachment, and noted the source IP of each.

iii. Identify senders. Used smtp contains FROM to pull sender addresses.

iv. Export and extract. Exported suspicious messages as .eml files, ran munpack to extract the MIME parts, and used tnef to open winmail.dat attachments.

v. Inspect attachments. Reviewed extracted images, patch files, text files, and C source for anything malicious or notable.

vi. Correlate across captures. Compared source IPs, subjects, and behavior across A to D to settle on one host.

## Phishing Email Analysis (C.pcap)

Three emails shared the same template: a claim that the recipient's computer was infected with remote-access malware and that they had been recorded through their webcam, an exact "password" shown as proof, and a demand for payment in Bitcoin. This is a common extortion (sextortion) scam. The passwords were included to make the threat seem credible.

| Subject line |	Quoted "password"|
| :--- | :--- |
| **Read carefully! - dayrit** | dayrit |
| **I can destroy everything! - 12345678** | 12345678 |
| **Don't wait too long! - fatima**	| fatima |

# Indicators of compromise

i. Sender display name "Your Life" with varying sender addresses

ii. Payment demand in BTC with a wallet address [TODO: add wallet from screenshot]

iii. Links to a Bitcoin purchase site and a wallet signup page presented as "help" for the victim

iv. Subject lines that embed the victim's password for urgency

# MITRE ATT&CK Mapping
| Observation | Technique |
| :--- | :--- |
| **Extortion emails sent to multiple recipients** |	T1566 Phishing |
| **Emails with attachments sent from 192.168.1.4 (A, D)** | T1566.001 Spearphishing Attachment |

# What I Learned
i. How SMTP traffic appears in Wireshark and how to isolate it with display filters

ii. How to recover attachments from raw packets, including Microsoft TNEF (winmail.dat) wrappers

iii. That the same host can appear in multiple captures, so correlation across files matters

iv. That extortion emails often rely on leaked or reused passwords rather than real access to the victim's machine

# Limitations and Next Steps
i. I identified the sender by IP address. In a real case I would also check email headers, timestamps, and mail server logs to confirm.

ii. I did not run the extracted files in a sandbox. Next time I would hash them and check them against VirusTotal.

iii. I would write this up as a short incident report with a timeline.

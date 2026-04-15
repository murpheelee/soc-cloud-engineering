# SOAR Playbook: Phishing Email Triage

## Trigger
- Email reported via phishing button (user submission)
- Email security gateway flags suspicious message

## Automation Level
Full auto-triage with analyst approval for containment actions.

## Workflow

```
Email Reported
     │
     ▼
┌─────────────┐
│ Extract IOCs │ ← URLs, attachments, sender domain, headers
└──────┬──────┘
       │
       ▼
┌──────────────┐
│ Enrich IOCs  │ ← VirusTotal, URLhaus, AbuseIPDB, Whois
└──────┬───────┘
       │
       ▼
┌──────────────────┐    ┌───────────────────┐
│ Verdict: Phish?  │───▶│ YES → Containment │
│ (Auto-scored)    │    └────────┬──────────┘
└────────┬─────────┘             │
         │ NO                    ▼
         ▼              ┌───────────────────┐
  Close as benign       │ Block sender      │
                        │ Purge from inboxes│
                        │ Isolate clickers  │
                        │ Reset passwords   │
                        └────────┬──────────┘
                                 │
                                 ▼
                        ┌───────────────────┐
                        │ Notify SOC + Mgmt │
                        │ Create incident   │
                        └───────────────────┘
```

## Step-by-Step

### Step 1: IOC Extraction (Automated)
| Action | Input | Output |
|--------|-------|--------|
| Parse email headers | Raw .eml file | Sender IP, return-path, SPF/DKIM/DMARC result |
| Extract URLs | Email body (HTML/text) | List of URLs, defanged |
| Extract attachments | Email MIME parts | File hashes (MD5, SHA256), file names |
| Identify recipients | Email headers | List of users who received the email |

### Step 2: IOC Enrichment (Automated)
| IOC Type | Enrichment Source | Verdict Logic |
|----------|------------------|---------------|
| URL | VirusTotal, URLhaus, Google Safe Browsing | Malicious if 3+ engines flag |
| File Hash | VirusTotal, Hybrid Analysis | Malicious if 5+ engines detect |
| Sender Domain | Whois, domain age check | Suspicious if domain < 30 days old |
| Sender IP | AbuseIPDB, Shodan | Suspicious if abuse score > 50 |

### Step 3: Auto-Scoring
```
Score = 0
IF sender domain age < 30 days:    score += 30
IF SPF/DKIM/DMARC fail:           score += 20
IF URL flagged by 3+ engines:      score += 40
IF attachment flagged by 5+ engines: score += 50
IF sender not in contacts:          score += 10

Verdict:
  score >= 50 → PHISHING (auto-contain)
  score 25-49 → SUSPICIOUS (analyst review)
  score < 25  → BENIGN (auto-close)
```

### Step 4: Containment (Analyst Approval Required)
1. Block sender domain at email gateway
2. Purge email from all recipient mailboxes
3. Block malicious URLs at web proxy
4. If any user clicked: isolate endpoint via EDR
5. If any user entered credentials: force password reset + revoke sessions

### Step 5: Notification
- SOC Team: Slack/Teams alert with full IOC summary
- Management: Email if > 10 users received the phish
- Affected Users: Auto-generated notification with guidance

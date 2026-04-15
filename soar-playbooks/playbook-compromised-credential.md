# SOAR Playbook: Compromised Credential Response

## Trigger
- Impossible travel alert (login from two distant locations)
- Brute force success (10+ failures followed by success)
- Credential found on dark web monitoring feed
- User reports unauthorized account activity

## Automation Level
Semi-automated — auto-disable account, analyst confirms scope before re-enabling.

## Workflow

```
Alert Triggered
     │
     ▼
┌──────────────────┐
│ Validate Alert   │ ← Check for known VPN, travel, shared accounts
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Disable Account  │ ← Immediate (automated)
│ Revoke Sessions  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Scope Assessment │ ← What did the attacker access?
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Remediate        │ ← Reset password, review MFA, check for persistence
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Re-enable Account│ ← After analyst confirmation
│ Monitor (30 days)│
└──────────────────┘
```

## Step-by-Step

### Step 1: Validate Alert (Automated)
| Check | Method | False Positive Criteria |
|-------|--------|----------------------|
| VPN Usage | Cross-reference with VPN logs | User connected to corporate VPN from both IPs |
| Known Travel | Check travel request system / calendar | User has approved travel to destination |
| Shared Account | Check if account is a shared/service account | Shared accounts have expected multi-location access |
| Timing | Evaluate time between logins | If > 12 hours apart, may be legitimate travel |

### Step 2: Immediate Containment (Automated)
1. Disable user account in Azure AD / Okta
2. Revoke all active sessions and refresh tokens
3. Revoke OAuth app consent grants
4. Block source IP at perimeter firewall (temporary 24h)

### Step 3: Scope Assessment (Analyst-Driven)
1. Review all sign-in logs for the past 72 hours
2. Check for inbox rules created (mail forwarding, auto-delete)
3. Review file access logs (SharePoint, OneDrive, S3)
4. Check for OAuth apps granted consent during compromise window
5. Check for MFA method changes or new device enrollments

### Step 4: Remediation
1. Force password reset (require new password at next login)
2. Re-register MFA (remove all existing methods, re-enroll)
3. Remove any suspicious inbox rules
4. Revoke any OAuth apps consented during compromise window
5. Review and remove any new devices enrolled

### Step 5: Account Recovery
- Analyst confirms remediation is complete
- Re-enable account with enhanced monitoring
- 30-day watch period with lowered alert thresholds
- User receives notification with security guidance

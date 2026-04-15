# Cloud Security Monitoring — AWS & Azure

## AWS Monitoring Coverage

### CloudTrail Events Monitored

| Category | Event Names | Detection Purpose |
|----------|------------|-------------------|
| IAM | CreateUser, CreateAccessKey, AttachUserPolicy | Unauthorized identity creation |
| S3 | PutBucketAcl, PutBucketPolicy, DeleteBucketPublicAccessBlock | Data exposure |
| EC2 | AuthorizeSecurityGroupIngress, RunInstances | Infrastructure changes |
| CloudTrail | StopLogging, DeleteTrail | Defense evasion |
| Lambda | CreateFunction, UpdateFunctionCode | Serverless abuse |
| Organizations | LeaveOrganization | Account takeover |

### AWS GuardDuty Integration

GuardDuty findings are forwarded to Splunk via CloudWatch Events → Lambda → HEC.

| Finding Type | Severity | SOAR Action |
|-------------|----------|-------------|
| UnauthorizedAccess:EC2/MaliciousIPCaller | High | Block IP, isolate instance |
| Recon:EC2/PortProbeUnprotectedPort | Medium | Alert SOC, review SG |
| CryptoCurrency:EC2/BitcoinTool | High | Terminate instance, investigate |
| Trojan:EC2/DNSDataExfiltration | Critical | Isolate instance, forensic capture |

## Azure Monitoring Coverage

### Azure Activity Log Events Monitored

| Category | Operation | Detection Purpose |
|----------|----------|-------------------|
| Authorization | Create/Delete role assignment | Privilege escalation |
| Security | Update security policy | Defense weakening |
| Network | Create/Update NSG rule | Network exposure |
| Compute | Create/Delete VM | Unauthorized compute |
| Storage | Set container ACL | Data exposure |
| Policy | Delete policy assignment | Compliance bypass |

### Defender for Cloud Integration

| Alert Type | Severity | SOAR Action |
|-----------|----------|-------------|
| Suspicious PowerShell activity | High | Isolate VM, capture memory |
| MFA disabled for user | Medium | Re-enable MFA, notify user |
| SSH brute force detected | High | Block source IP, alert SOC |
| Anomalous Azure AD sign-in | Medium | Conditional access block |

## Cross-Cloud Correlation Rules

| Rule | AWS Source | Azure Source | Detection Logic |
|------|-----------|-------------|----------------|
| Same attacker IP across clouds | CloudTrail sourceIP | Azure SignIn ipAddress | Alert when same IP triggers events in both clouds within 1 hour |
| Credential reuse | IAM CreateAccessKey | Azure AD new app credential | Same user creates new credentials in both clouds within 24 hours |
| Coordinated defense evasion | StopLogging | Delete diagnostic settings | Logging disabled in both clouds within same time window |

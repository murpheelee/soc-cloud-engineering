# SOAR Playbook: Cloud Resource Public Exposure

## Trigger
- S3 bucket ACL changed to public
- Azure Blob container set to anonymous access
- Security group modified to allow 0.0.0.0/0 on sensitive ports
- Cloud Storage public access block removed

## Automation Level
Full auto-remediate with notification — no analyst approval needed due to clear policy violation.

## Workflow

```
Alert Triggered
     │
     ▼
┌──────────────────┐
│ Identify Resource│ ← Bucket name, SG ID, region, account
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Auto-Remediate   │ ← Restore secure configuration
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Verify Fix       │ ← Confirm resource is no longer public
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ Notify & Document│ ← Alert resource owner + SOC + create ticket
└──────────────────┘
```

## Remediation Actions by Resource Type

### S3 Bucket Made Public
| Step | Action | API Call |
|------|--------|---------|
| 1 | Re-enable public access block | `aws s3api put-public-access-block --bucket <name> --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true` |
| 2 | Remove public ACL grants | `aws s3api put-bucket-acl --bucket <name> --acl private` |
| 3 | Verify | `aws s3api get-public-access-block --bucket <name>` |

### Security Group Open to Internet
| Step | Action | API Call |
|------|--------|---------|
| 1 | Revoke 0.0.0.0/0 ingress rule | `aws ec2 revoke-security-group-ingress --group-id <sg-id> --protocol tcp --port <port> --cidr 0.0.0.0/0` |
| 2 | Verify | `aws ec2 describe-security-groups --group-ids <sg-id>` |

### Azure Blob Container Public Access
| Step | Action | CLI Command |
|------|--------|-------------|
| 1 | Set container to private | `az storage container set-permission --name <container> --public-access off` |
| 2 | Verify | `az storage container show-permission --name <container>` |

## Notification Template

```
Subject: [AUTO-REMEDIATED] Cloud Resource Public Exposure — {resource_name}

A cloud resource was detected as publicly accessible and has been
automatically remediated.

Resource:     {resource_type} — {resource_name}
Account:      {cloud_account}
Region:       {region}
Changed By:   {actor_arn}
Detected At:  {timestamp}
Remediated:   {remediation_timestamp}

Action Taken: {remediation_action}

If this change was intentional, please submit a security exception
request through the standard process.

— SOC Automation
```

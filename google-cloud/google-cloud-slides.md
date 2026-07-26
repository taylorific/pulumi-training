---
layout: section
---

# Google Cloud

<br>
<br>
<Link to="toc" title="Table of Contents"/>

---
hideInToc: true
---

## Bootstrap Pulumi on Google Cloud

Manually do this:

1. Create/verify Google Workspace or Cloud Identity.
2. Verify your Google Cloud organization exists.
3. Create one bootstrap project.
4. Link billing.
5. Enable a few APIs.
6. Create a Pulumi service account.
7. Grant your human admin permission to impersonate that service account.

After that, Pulumi can manage almost everything.

---
hideInToc: true
---

## Service Account Impersonation

Google Cloud recommends service account impersonation instead of long-lived service account keys because impersonation uses short-lived credentials and requires a prior authenticated identity.  
https://docs.cloud.google.com/docs/authentication/use-service-account-impersonation

Pulumi’s GCP provider supports service account impersonation through `impersonateServiceAccount`.
https://www.pulumi.com/registry/packages/gcp/installation-configuration

---
hideInToc: true
---

## Google Cloud Native Provider currently paused

The GCP (classic) provider is maintained directly by Google themselves and has recently become something they're providing as a primary development experience, resulting in a provider of high quality and completeness.

The preview of the Google Native provider has highlighted a number of challenges due to how GCP's API are designed. There's more use of imperative configuration methods (i.e. POST requests) to affect the values of specific properties of the resource. These are not always described in a formal method to indicate the property they affect. This leaves certain areas of the resources inaccessible.

https://github.com/pulumi/pulumi/discussions/12470

---

# Google Cloud Organization Setup

Choosing between **Cloud Identity** and **Google Workspace**

---
layout: section

# Why an Organization?

---

# What is a Google Cloud Organization?

An Organization is the **root resource** in Google Cloud.

Everything lives underneath it.

```text
Organization
│
├── Folders (optional)
│   ├── Engineering
│   ├── Production
│   └── Development
│
└── Projects
    ├── dev-network
    ├── production
    └── shared-services
```

Benefits:

- Centralized IAM
- Billing management
- Organization Policies
- Shared VPC
- Centralized logging
- Service Accounts

---

# What Creates an Organization?

A Google Cloud Organization is created when you verify ownership of a domain through:

- Google Workspace
- Cloud Identity

Example:

```
example.com
```

becomes

```
Organization
Display Name: example.com
```

---

layout: section

# Identity Options

---

# Two Ways to Get an Organization

| Cloud Identity | Google Workspace |
|----------------|------------------|
| Identity only | Identity + Productivity |
| Free edition available | Paid subscription |
| Google Cloud IAM | Google Cloud IAM |
| No Gmail | Gmail included |
| No Drive | Drive included |
| No Docs | Docs included |

Both create the exact same Google Cloud Organization.

---

# Cloud Identity

Cloud Identity provides:

- Users
- Groups
- MFA
- SSO
- Directory
- Google Cloud IAM

Users can:

- Login to Google Cloud Console
- Use gcloud
- Run Terraform/Pulumi
- Access APIs

Users **cannot**:

- Send Gmail
- Use Google Docs
- Use Drive
- Use Calendar

---

# Google Workspace

Workspace includes everything in Cloud Identity plus:

- Gmail
- Drive
- Docs
- Sheets
- Calendar
- Meet
- Chat

Think of Workspace as:

```
Google Workspace
    =
Cloud Identity
    +
Google Productivity Apps
```

---

# Comparison

| Feature | Cloud Identity | Workspace |
|----------|---------------|-----------|
| Google Cloud Console | ✅ | ✅ |
| IAM | ✅ | ✅ |
| Groups | ✅ | ✅ |
| MFA | ✅ | ✅ |
| Gmail | ❌ | ✅ |
| Drive | ❌ | ✅ |
| Calendar | ❌ | ✅ |
| Docs | ❌ | ✅ |

---

layout: section

# Which Should I Choose?

---

# Cloud Identity Free

Good choice when:

- You only need Google Cloud
- Your company already uses Microsoft 365
- You don't want Gmail
- You only need identities

Typical users:

```
admin@example.com
developer@example.com
contractor@example.com
```

---

# Google Workspace

Best choice when:

- Company email is Gmail
- You use Google Docs
- You use Google Meet
- You use Google Calendar

The identity system is exactly the same as Cloud Identity.

---

# Common Misconception

Many people see:

```
Cloud Identity
$7.20/user/month
```

This is **Cloud Identity Premium**.

There are actually two editions:

| Edition | Cost |
|----------|------|
| Cloud Identity Free | Free |
| Cloud Identity Premium | Paid |

Premium adds enterprise identity features.

Google Cloud does **not** require Premium.

---

layout: section

# Users vs Service Accounts

---

# Human Users

Human administrators belong to your identity provider.

Examples:

```
alice@example.com
bob@example.com
admin@example.com
```

These users:

- Login to the Cloud Console
- Use gcloud auth login
- Receive IAM roles

---

# Service Accounts

Service Accounts are **not** Cloud Identity users.

Example:

```
pulumi@example-project.iam.gserviceaccount.com
```

Characteristics:

- Cannot login interactively
- No Gmail
- No password
- Used by automation
- Managed entirely by IAM

---
hideInToc: true
---

# Login to Google Cloud and get your organization ID

```bash
docker run -ti \
  --name gcloud-config \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:stable \
    gcloud auth login
docker run --rm \
  --volumes-from gcloud-config \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:stable \
    gcloud organizations list
ORG_ID=$(docker run --rm \
  --volumes-from gcloud-config \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:stable \
    gcloud organizations list --format='value(ID)')
```

---
hideInToc: true
---

# Create the bootstrap project

```bash
# Need to make uniuqe
SUFFIX=$(openssl rand -hex 2)
PROJECT_ID=pulumi-bootstrap-${SUFFIX}
docker run --rm \
  --env PROJECT_ID \
  --env ORG_ID \
  --volumes-from gcloud-config \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:stable \
    gcloud projects create "$PROJECT_ID" \
      --organization="$ORG_ID" \
      --name="pulumi-bootstrap"
```

---
hideInToc: true
---

```
docker run --rm \
  --volumes-from gcloud-config \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:stable \
    gcloud billing accounts list
BILLING_ACCOUNT=$(docker run --rm \
  --volumes-from gcloud-config \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:stable \
    gcloud billing accounts list \
      --filter="OPEN=True" \
      --format="value(ACCOUNT_ID)")
```

---
hideInToc: true
---

```
docker run --rm \
  --volumes-from gcloud-config \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:stable \
    gcloud billing projects link "$PROJECT_ID" \
      --billing-account="$BILLING_ACCOUNT"
```

---
hideInToc: true
---

# Enable APIs

```
docker run --rm \
  --volumes-from gcloud-config \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:stable \
    gcloud services enable \
    cloudresourcemanager.googleapis.com \
    iam.googleapis.com \
    serviceusage.googleapis.com \
    cloudbilling.googleapis.com \
    --project "$PROJECT_ID"
```

---
hideInToc: true
---

# Create Pulumi Service Account

```
SA_NAME="pulumi-admin"
docker run --rm \
  --env SA_NAME \
  --volumes-from gcloud-config \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:stable \
    gcloud iam service-accounts create "$SA_NAME" \
    --project "$PROJECT_ID" \
    --display-name="Pulumi Admin"
```

---
hideInToc: true
---

# Grant Pulumi Service Account Org Level Permissions

```
SA_NAME="pulumi-admin"
SA_EMAIL="${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"
docker run --rm \
  --env ORG_ID --env SA_EMAIL \
  --volumes-from gcloud-config \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:stable \
    gcloud organizations add-iam-policy-binding "$ORG_ID" \
      --member="serviceAccount:${SA_EMAIL}" \
      --role="roles/resourcemanager.organizationAdmin"

docker run --rm \
  --env ORG_ID --env SA_EMAIL \
  --volumes-from gcloud-config \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:stable \
    gcloud organizations add-iam-policy-binding "$ORG_ID" \
      --member="serviceAccount:${SA_EMAIL}" \
      --role="roles/resourcemanager.projectCreator"

docker run --rm \
  --env ORG_ID --env SA_EMAIL \
  --volumes-from gcloud-config \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:stable \
    gcloud organizations add-iam-policy-binding "$ORG_ID" \
      --member="serviceAccount:${SA_EMAIL}" \
      --role="roles/billing.user"
```

---
hideInToc: true
---

# Let human admin impersonate service account

```
HUMAN_EMAIL=alice@example.com
docker run --rm \
  --env PROJECT_ID --env HUMAN_EMAIL --env SA_EMAIL \
  --volumes-from gcloud-config \
  gcr.io/google.com/cloudsdktool/google-cloud-cli:stable \
    gcloud iam service-accounts add-iam-policy-binding "$SA_EMAIL" \
      --project "$PROJECT_ID" \
      --member="user:${HUMAN_EMAIL}" \
      --role="roles/iam.serviceAccountTokenCreator"
```

---
hideInToc: true
---

# Run Pulumi with impersonation

```
docker run -it --rm \
  --env SA_EMAIL \
  --env PROJECT_ID \
  --entrypoint /bin/bash \
  pulumi/pulumi:latest

  gcloud auth login --no-launch-browser
  gcloud auth application-default login --no-launch-browser
  pulumi config set gcp:project "$PROJECT_ID"
  pulumi config set gcp:impersonateServiceAccount "$SA_EMAIL"
```

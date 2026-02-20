# AWS Static Website on AWS (S3 + CloudFront + Route 53 + ACM + CloudWatch) + CI/CD

**Live:** https://adonyas.com  
**Repo:** https://github.com/adonyasa03-byte/AWS-Static-Website

This project is my personal portfolio website deployed using a production-style AWS setup: a private S3 origin behind CloudFront with a custom domain, HTTPS, monitoring/alerts, and GitHub Actions auto-deploy.

---

## What I Built (Recruiter Summary)

- **Global CDN website** served by **CloudFront**
- **Private S3 bucket** origin (no public access) secured with **Origin Access Control (OAC)**
- **Custom domain + HTTPS** using **Route 53** + **ACM**
- **Monitoring + alerting** using **CloudWatch dashboard + alarms + SNS email notifications**
- **CI/CD**: push to `main` automatically deploys to S3 and invalidates CloudFront so changes show fast

---

## Services Used

- **S3** (origin bucket): `adonyasa-website` (Region: `us-west-1`)
- **CloudFront** (Distribution): `E3TE1D75SFWYOB`
- **Route 53** (DNS): custom domain records pointing to CloudFront
- **ACM**: public certificate used by CloudFront for HTTPS
- **CloudWatch**: dashboard + alarms
- **SNS**: email notifications for alarms
- **GitHub Actions**: automated deploy + CloudFront invalidation

---

## Key Decisions / Best Practices

- **No public S3 website hosting**: S3 stays private; CloudFront handles all user traffic.
- **OAC instead of public bucket**: CloudFront signs requests to access S3 securely.
- **HTTPS enforced**: CloudFront redirects HTTP to HTTPS.
- **Fast updates**: CloudFront cache policy set to **CachingDisabled** (so updates to `index.html` appear quickly without waiting on TTLs).
- **Operational readiness**: CloudWatch alarms + SNS email for 4xx/5xx error conditions.

---

## Deployment Architecture

Users → **CloudFront (HTTPS)** → **S3 (private bucket via OAC)**  
DNS: **Route 53** aliases domain → CloudFront  
Certificate: **ACM** attached to CloudFront

---

## Step-by-step Deployment (High Level)

### 1) Register domain / set up hosted zone (Route 53)
- Register/host the domain in Route 53
- Ensure a hosted zone exists for the domain

### 2) Create S3 bucket for website content (private)
Create a bucket (example: `adonyasa-website` in `us-west-1`):
- Block all public access: **ON**
- Versioning: **ON**
- Default encryption: **ON** (SSE-S3)
- No S3 static website hosting required

### 3) Upload website files
- Upload `index.html` (keep private)

### 4) Request SSL certificate (ACM)
- Request a public certificate for the domain (and optional `www`)
- Validate using DNS
- Attach the certificate to the CloudFront distribution

### 5) Create/attach Origin Access Control (OAC)
- Create OAC (S3 origin type, sign requests)
- Attach to the CloudFront distribution and keep S3 private

### 6) Create CloudFront distribution
- Origin: S3 bucket
- Viewer protocol policy: **Redirect HTTP to HTTPS**
- Allowed methods: **GET, HEAD**
- Cache policy: **CachingDisabled**
- Default root object: **index.html**
- Alternate domain names: your domain
- SSL certificate: from ACM

### 7) Route 53 DNS configuration
Create alias records pointing to CloudFront:
- `adonyas.com` → Alias A (and AAAA optional) → CloudFront distribution  
- `www.adonyas.com` (optional) → Alias A (and AAAA optional) → CloudFront distribution

---

## Monitoring & Alerting (CloudWatch + SNS)

### Dashboard
CloudWatch dashboard includes CloudFront per-distribution metrics such as:
- Requests
- BytesDownloaded
- 4xxErrorRate
- 5xxErrorRate
- TotalErrorRate

### Alarms + SNS Email
Alarms are configured to send email via SNS when:
- **4xxErrorRate** exceeds a threshold (client-side errors: bad links, unauthorized, etc.)
- **5xxErrorRate** exceeds a threshold (server-side / origin issues)

---

## CI/CD: GitHub Actions Auto-Deploy

On every push to `main`, GitHub Actions:
1. Syncs repo contents to S3 (`adonyasa-website`)
2. Creates a CloudFront invalidation so updates show quickly

### Required GitHub Secrets
Repo → Settings → Secrets and variables → Actions:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_REGION` = `us-west-1`
- `S3_BUCKET` = `adonyasa-website`
- `CLOUDFRONT_DISTRIBUTION_ID` = `E3TE1D75SFWYOB`

---

## How to Update the Site

1. Edit `index.html`
2. Commit and push to `main`
3. GitHub Actions deploys automatically and invalidates CloudFront

---

## Future Improvements (Not Implemented Yet)

- CloudFront/S3 access logging to a dedicated logs bucket
- Backup/replication bucket for the website origin
- AWS WAF Web ACL for additional edge protection

---

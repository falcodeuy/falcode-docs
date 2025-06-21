---
layout: default
title: Django storages (with R2)
parent: Project configuration
grand_parent: Django
nav_order: 5
---

# Set Up Django Storages and Cloudflare R2

## 1. Create and Configure a Cloudflare R2 Bucket

### Step 1: Access Cloudflare Dashboard

* Log in to your [Cloudflare dashboard](https://dash.cloudflare.com).
* Select your account and go to **R2** under the "Storage" section.

### Step 2: Create an R2 Bucket

* Click **+ Create Bucket**.
* Provide a unique bucket name.
* Choose your preferred region (optional).
* Choose storage class **Standard**.
* Click **Create Bucket**.

### Step 3: Configure Access Keys

* Go to **API Tokens** or **R2 Keys** section.
* Create an **Access Key ID** and **Secret Access Key**.
* Save these credentials securely for programmatic access.

---

## 2. Set Up a Public Domain for Your R2 Bucket

### For Domains Managed in Cloudflare (DNS Zone in Cloudflare)

* In the Cloudflare dashboard, go to your **R2 bucket settings**.
* Under the **Custom Domains** section, add your desired domain or subdomain (e.g., `images.example.com`).
* Cloudflare will automatically create the required DNS records for you.
* This sets up the domain to serve content from your R2 bucket via Cloudflare’s CDN and proxy.

### For Domains NOT Managed in Cloudflare (External DNS Zone)

* To use a custom domain for R2 with DNS outside Cloudflare, you need a **Business Plan** or higher.
* This enables **Partial CNAME Setup** allowing you to proxy subdomains via Cloudflare without full zone transfer.
* More info in Cloudflare official docs:
  [Partial (CNAME) setup](https://developers.cloudflare.com/dns/zone-setups/partial-setup/)

---

## 3. Configure Django to Use Cloudflare R2 with `django-storages`

You can use `django-storages` with the **S3 backend** configured to point to Cloudflare R2, which is fully S3-compatible.

### Step 1: Install dependencies

```bash
pip install django-storages boto3
```

### Step 2: Update your `settings.py`

```python
INSTALLED_APPS = [
    # ...
    'storages',
]

# Use custom storage backend for media files
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'

# AWS S3 compatible settings for Cloudflare R2
AWS_ACCESS_KEY_ID = '<YOUR_R2_ACCESS_KEY_ID>'
AWS_SECRET_ACCESS_KEY = '<YOUR_R2_SECRET_ACCESS_KEY>'
AWS_STORAGE_BUCKET_NAME = '<YOUR_R2_BUCKET_NAME>'
AWS_S3_ENDPOINT_URL = 'https://<accountid>.r2.cloudflarestorage.com'  # Replace <accountid>
AWS_S3_REGION_NAME = 'auto'  # R2 ignores this but boto3 requires a value
AWS_S3_ADDRESSING_STYLE = "virtual"  # Use virtual-hosted style addressing

# Optional: configure URL for serving media files via your custom domain (Cloudflare proxy)
MEDIA_URL = 'https://images.example.com/'  # Your custom CNAME domain configured in Cloudflare DNS

# Optional: additional boto3 settings for improved compatibility
AWS_S3_SIGNATURE_VERSION = 's3v4'
```

---

## 4. Migrate Images from AWS S3 or EC2 Server to Cloudflare R2

### Migrating from AWS S3

1. Use AWS CLI to download:

```bash
aws s3 sync s3://source-bucket-name ./local-folder
```

2. Upload to R2 using **rclone** configured with R2 credentials:

```bash
rclone copy ./local-folder r2:your-r2-bucket-name --s3-endpoint=https://<accountid>.r2.cloudflarestorage.com
```

### Migrating from EC2 Server

1. Copy images from EC2 instance to local:

```bash
scp -r ubuntu@ec2-ip:/path/to/images ./local-folder
```

2. Upload to R2 using **rclone**:

```bash
rclone copy ./local-folder r2:your-r2-bucket-name --s3-endpoint=https://<accountid>.r2.cloudflarestorage.com
```

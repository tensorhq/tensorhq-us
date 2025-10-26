# tensorhq.us

Minimal static website for tensorhq.us

## Deployment to AWS S3 + CloudFront + Route53

### Prerequisites
- AWS CLI installed and configured
- Domain registered (tensorhq.us)
- AWS account with appropriate permissions

### Step 1: Create S3 Bucket

```bash
# Create S3 bucket (use us-east-1 for CloudFront compatibility)
aws s3 mb s3://tensorhq.us --region us-east-1

# Enable static website hosting
aws s3 website s3://tensorhq.us --index-document index.html --error-document index.html
```

### Step 2: Upload Website Files

```bash
# Sync files to S3
aws s3 sync . s3://tensorhq.us --exclude ".git/*" --exclude "README.md" --exclude ".gitignore"

# Set files to public-read (if using S3 website endpoint)
aws s3 sync . s3://tensorhq.us --exclude ".git/*" --exclude "README.md" --exclude ".gitignore" --acl public-read
```

### Step 3: Create CloudFront Distribution

1. Go to CloudFront in AWS Console
2. Create new distribution with:
   - Origin Domain: `tensorhq.us.s3.amazonaws.com`
   - Viewer Protocol Policy: Redirect HTTP to HTTPS
   - Alternate Domain Names (CNAMEs): `tensorhq.us`, `www.tensorhq.us`
   - SSL Certificate: Request or import certificate via ACM (must be in us-east-1)
   - Default Root Object: `index.html`

Or via CLI (after creating SSL certificate in ACM):

```bash
aws cloudfront create-distribution --origin-domain-name tensorhq.us.s3.amazonaws.com \
  --default-root-object index.html
```

### Step 4: Configure Route53

1. Go to Route53 in AWS Console
2. Create/Select hosted zone for `tensorhq.us`
3. Create A record:
   - Record name: (leave empty for root domain)
   - Record type: A
   - Alias: Yes
   - Alias target: Select CloudFront distribution
4. Create A record for `www`:
   - Record name: www
   - Record type: A
   - Alias: Yes
   - Alias target: Select CloudFront distribution

### Step 5: Request SSL Certificate (ACM)

```bash
# Request certificate (must be in us-east-1 for CloudFront)
aws acm request-certificate \
  --domain-name tensorhq.us \
  --subject-alternative-names www.tensorhq.us \
  --validation-method DNS \
  --region us-east-1
```

Then add the DNS validation records to Route53.

### Updates

To update the website:

```bash
# Upload new files
aws s3 sync . s3://tensorhq.us --exclude ".git/*" --exclude "README.md" --exclude ".gitignore" --acl public-read

# Invalidate CloudFront cache
aws cloudfront create-invalidation --distribution-id YOUR_DISTRIBUTION_ID --paths "/*"
```

## Local Development

Simply open `index.html` in a browser to preview locally.

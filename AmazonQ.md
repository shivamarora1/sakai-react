# Deploying Next.js to Amazon S3

This document outlines the setup for deploying this Next.js application to Amazon S3 using GitHub Actions.

## Prerequisites

Before the GitHub Actions workflow can successfully deploy to S3, you need to:

1. Create an S3 bucket for hosting the website
2. Configure the S3 bucket for static website hosting
3. Set up proper IAM permissions
4. Add GitHub repository secrets

## S3 Bucket Setup

1. Create an S3 bucket with a name of your choice
2. Enable static website hosting:
   - Go to the bucket properties
   - Enable "Static website hosting"
   - Set "Index document" to `index.html`
   - Set "Error document" to `404.html` or `index.html`
3. Configure bucket permissions:
   - Update the bucket policy to allow public read access (if this is a public website)

Example bucket policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

## IAM User Setup

1. Create an IAM user with programmatic access
2. Attach a policy with permissions to write to your S3 bucket:

Example IAM policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::YOUR-BUCKET-NAME",
        "arn:aws:s3:::YOUR-BUCKET-NAME/*"
      ]
    }
  ]
}
```

## GitHub Repository Secrets

Add the following secrets to your GitHub repository:

1. `AWS_ACCESS_KEY_ID`: Your IAM user's access key
2. `AWS_SECRET_ACCESS_KEY`: Your IAM user's secret key
3. `AWS_REGION`: The AWS region where your S3 bucket is located (e.g., `us-east-1`)
4. `S3_BUCKET_NAME`: The name of your S3 bucket

If using CloudFront:
5. `CLOUDFRONT_DISTRIBUTION_ID`: Your CloudFront distribution ID

## Next.js Configuration

The `next.config.js` file has been updated to enable static HTML export:

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export',  // Enables static HTML export
  images: {
    unoptimized: true  // Required for static export
  }
}

module.exports = nextConfig
```

## Workflow File

The GitHub Actions workflow file is located at `.github/workflows/deploy-to-s3.yml`. It will:

1. Build the Next.js application
2. Deploy the built files to your S3 bucket
3. Optionally invalidate CloudFront cache if configured

## Accessing Your Website

After deployment, your website will be available at:
- S3 website URL: `http://YOUR-BUCKET-NAME.s3-website-YOUR-REGION.amazonaws.com`
- Or your custom domain if configured with CloudFront

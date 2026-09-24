# Frontend Design

## Overview

The React application is built into static files and served from S3 through CloudFront.

```plaintext
User browser
   |
   v
CloudFront
   |
   v
S3 bucket with React build assets
```

## S3

The S3 bucket stores the production build output, usually files such as `index.html`, JavaScript bundles, CSS and images.

- Block all public access.
- Allow access only through CloudFront.
- Enable bucket versioning.
- Use lifecycle rules for old build artifacts if needed.

The bucket should not be used as a public website endpoint. CloudFront should be the public entry point.

## CloudFront

CloudFront improves performance by caching static assets near users and terminating TLS at the edge.

- Cache fingerprinted static assets for a long time.
- Keep `index.html` cache time short so deployments roll out cleanly.
- Redirect HTTP traffic to HTTPS.
- Use AWS Certificate Manager for the custom domain certificate.
- Route SPA deep links back to `index.html`.

## Deployment Flow

```text
Developer push
   |
   v
CI/CD pipeline
   |
   v
Build React app
   |
   v
Upload files to S3
   |
   v
Create CloudFront invalidation for changed entry files
```

Only the frontend build artifacts should be uploaded. Source files, environment templates and local config files should not be copied into the bucket.

## Environment Configuration

Frontend configuration should be handled carefully because anything shipped to the browser is public.

- Safe to expose: API base URL, public analytics identifiers and non-sensitive feature flags.
- Not safe to expose: secrets, tokens or passwords.

---
outline: deep
---

# Deployment configuration

This guide explains how to configure open-dpp for a hosted deployment. The application is configured with environment variables. Do not commit real secrets or credentials to the repository.

## Required services

A hosted deployment normally needs:

- A Node.js/Docker host for the open-dpp application
- MongoDB for application data
- S3-compatible object storage for DPP files and profile pictures
- An SMTP provider for email delivery

For a small demo deployment, these can be hosted with Render, MongoDB Atlas, Cloudflare R2, and an SMTP provider such as Brevo.

## Environment variables

### Application

| Variable | Example | Description |
| --- | --- | --- |
| `OPEN_DPP_PORT` | `10000` | Port used by the application. Set this to the port required by your hosting provider. |
| `OPEN_DPP_URL` | `https://open-dpp.onrender.com` | Public origin of the application. Do not include `/api`, a path, query string, or fragment. |
| `OPEN_DPP_LOG_FORMAT` | `json` | Optional log format: `plain` or `json`. |
| `OPEN_DPP_BUILD_API_DOC` | `false` | Whether to enable generated API documentation. |

`OPEN_DPP_URL` must be the public origin only:

```env
OPEN_DPP_URL=https://open-dpp.example.com
```

Do not use:

```env
OPEN_DPP_URL=https://open-dpp.example.com/api
```

The application generates the frontend API URL by appending `/api` to `OPEN_DPP_URL`.

### MongoDB

Use either a complete connection URI or the individual host, port, username, and password settings. A hosted deployment should normally use `OPEN_DPP_MONGODB_URI`.

```env
OPEN_DPP_MONGODB_URI=mongodb+srv://USER:PASSWORD@CLUSTER.mongodb.net/management
```

When using a URI, the individual MongoDB host and port variables are not required.

If using individual settings instead:

```env
OPEN_DPP_MONGODB_HOST=mongodb.example.com
OPEN_DPP_MONGODB_PORT=27017
OPEN_DPP_MONGODB_USER=open_dpp
OPEN_DPP_MONGODB_PASSWORD=CHANGE_ME
OPEN_DPP_MONGODB_DATABASE=management
```

URL-encode special characters in MongoDB usernames and passwords when they appear inside a connection URI.

### S3-compatible object storage

open-dpp uses an S3-compatible client for uploaded product files and profile pictures. The endpoint is configured as a hostname without a protocol.

```env
OPEN_DPP_S3_ENDPOINT=ACCOUNT_ID.r2.cloudflarestorage.com
OPEN_DPP_S3_PORT=443
OPEN_DPP_S3_SSL=true
OPEN_DPP_S3_ACCESS_KEY=YOUR_ACCESS_KEY
OPEN_DPP_S3_SECRET_KEY=YOUR_SECRET_KEY
OPEN_DPP_S3_DEFAULT_BUCKET=open-dpp-files
OPEN_DPP_S3_PROFILE_PICTURE_BUCKET=open-dpp-profile-pictures
```

Create both buckets before starting the application. The application expects the bucket names configured in these variables to exist.

For local RustFS or MinIO:

```env
OPEN_DPP_S3_ENDPOINT=localhost
OPEN_DPP_S3_PORT=9000
OPEN_DPP_S3_SSL=false
OPEN_DPP_S3_ACCESS_KEY=rustfsadmin
OPEN_DPP_S3_SECRET_KEY=CHANGE_ME
```

### Email

The local Docker Compose setup uses Mailpit, but a hosted deployment needs a reachable SMTP provider.

Example SMTP configuration:

```env
OPEN_DPP_MAIL_HOST=smtp.example.com
OPEN_DPP_MAIL_PORT=587
OPEN_DPP_MAIL_USER=YOUR_SMTP_USERNAME
OPEN_DPP_MAIL_PASSWORD=YOUR_SMTP_PASSWORD
OPEN_DPP_MAIL_SENDER_ADDRESS=verified@example.com
```

Use the SMTP port and encryption settings required by your provider. Verify the sender address with the provider before sending email to users.

### Authentication

Set a long, random secret for authentication:

```env
OPEN_DPP_AUTH_SECRET=GENERATE_A_LONG_RANDOM_SECRET
```

Generate one locally with:

```bash
openssl rand -base64 48
```

Never use the example value in production and never commit the secret to Git.

### AI integration

The Mistral API key is used by AI-related functionality:

```env
OPEN_DPP_MISTRAL_API_KEY=YOUR_MISTRAL_API_KEY
```

Leave this unset if AI features are not needed.

### Optional virus scanning

Virus scanning is disabled when `OPEN_DPP_CLAMAV_URL` is unset. To use a deployed ClamAV REST service:

```env
OPEN_DPP_CLAMAV_URL=https://clamav.example.com
```

The value must be a complete HTTP or HTTPS URL.

### Optional OAuth provider

To enable the open-dpp OAuth/OIDC provider, configure all required client values:

```env
OPEN_DPP_OAUTH_PROVIDER_ENABLED=true
OPEN_DPP_OAUTH_PROVIDER_CLIENT_ID=landing-page
OPEN_DPP_OAUTH_PROVIDER_CLIENT_SECRET=CHANGE_ME
OPEN_DPP_OAUTH_PROVIDER_REDIRECT_URIS=https://landing.example.com/auth/callback
OPEN_DPP_OAUTH_PROVIDER_CLIENT_NAME=Landing page
```

Keep the provider disabled unless you have configured a trusted client and redirect URI.

### Instance settings

These optional settings control signup and organization creation:

```env
OPEN_DPP_INSTANCE_SIGNUP_ENABLED=true
OPEN_DPP_INSTANCE_ORGANIZATION_CREATION_ENABLED=true
```

Use the strings `true` or `false`.

## Render configuration example

For a Docker-based Render web service, use:

```env
OPEN_DPP_PORT=10000
OPEN_DPP_URL=https://open-dpp.onrender.com
OPEN_DPP_LOG_FORMAT=json
OPEN_DPP_BUILD_API_DOC=false

OPEN_DPP_MONGODB_URI=mongodb+srv://USER:PASSWORD@CLUSTER.mongodb.net/management

OPEN_DPP_S3_ENDPOINT=ACCOUNT_ID.r2.cloudflarestorage.com
OPEN_DPP_S3_PORT=443
OPEN_DPP_S3_SSL=true
OPEN_DPP_S3_ACCESS_KEY=YOUR_ACCESS_KEY
OPEN_DPP_S3_SECRET_KEY=YOUR_SECRET_KEY
OPEN_DPP_S3_DEFAULT_BUCKET=open-dpp-files
OPEN_DPP_S3_PROFILE_PICTURE_BUCKET=open-dpp-profile-pictures

OPEN_DPP_MAIL_HOST=smtp.example.com
OPEN_DPP_MAIL_PORT=587
OPEN_DPP_MAIL_USER=YOUR_SMTP_USERNAME
OPEN_DPP_MAIL_PASSWORD=YOUR_SMTP_PASSWORD
OPEN_DPP_MAIL_SENDER_ADDRESS=verified@example.com

OPEN_DPP_AUTH_SECRET=GENERATE_A_LONG_RANDOM_SECRET
OPEN_DPP_MISTRAL_API_KEY=YOUR_MISTRAL_API_KEY
```

Configure these values in the hosting provider's environment-variable settings rather than committing a `.env` file.

## Local development configuration

Copy the example configuration before starting local development:

```bash
cp .env.dev.example .env.dev
cp .env.example .env
```

Review the values and start the local services:

```bash
docker compose up -d
pnpm install
pnpm dev
```

The local Docker Compose configuration provides MongoDB, RustFS, and Mailpit for development. These services should not be exposed publicly without authentication and additional security controls.

## Security checklist

- Use strong, unique passwords for MongoDB, object storage, SMTP, and OAuth.
- Store secrets in the hosting provider's secret manager or environment settings.
- Do not commit `.env`, `.env.dev`, credentials, API keys, or private certificates.
- Restrict MongoDB network access where possible.
- Restrict object-storage API tokens to the required buckets and permissions.
- Use HTTPS for the public application URL and production integrations.
- Configure a verified sender address for email.
- Back up MongoDB data and uploaded object-storage files separately from the Git repository.

## Troubleshooting

### The application exits during startup

Check that every required variable is present. In particular, verify `OPEN_DPP_URL`, the S3 variables, the SMTP variables, and `OPEN_DPP_AUTH_SECRET`.

### MongoDB connection fails

Check the connection string, URL-encode special characters in credentials, and allow the hosting service to connect in the MongoDB network settings.

### File uploads fail

Check that both configured buckets exist, the S3 endpoint does not include `https://`, the port is correct, and SSL is set to `true` for hosted HTTPS object storage.

### Emails do not arrive

Check the SMTP host, port, username, password, verified sender, and provider logs. Mailpit is intended for local development and does not deliver mail to real users.

### Browser reports a CORS error

The backend must allow the deployed application origin. Set the frontend/backend origin consistently with `OPEN_DPP_URL` and redeploy after changing environment variables.

---
outline: deep
---

# Deployment technology stack and cost comparison

This document describes the recommended deployment architecture for open-dpp, an application built with Vue, TypeScript, NestJS, MongoDB, S3-compatible object storage, email delivery, and optional AI integration.

The recommended setup is designed for a project that does not need to be available continuously, while still preserving application data when the web service sleeps or restarts.

## Executive recommendation

For a student project, prototype, or low-traffic demonstration, use:

```text
Render Free
MongoDB Atlas Free
Cloudflare R2 Standard
HTTPS email provider or email disabled during testing
```

Estimated infrastructure cost:

```text
₹0/month within the free limits
```

For important data that needs better storage and backups, use:

```text
Render Free
MongoDB Atlas Flex
Cloudflare R2 Standard
HTTPS email provider
```

Estimated base cost:

```text
Atlas Flex: $8–$30/month
Approximately ₹720–₹2,700/month at a planning rate of ₹90 per US dollar
```

The final INR amount depends on the exchange rate, taxes, payment-provider charges, and actual usage. Confirm the price in each provider's dashboard before purchasing.

## Existing project architecture

The repository contains a monorepo with:

- `apps/client`: Vue frontend built with Vite.
- `apps/main`: NestJS backend and API server.
- `packages/*`: Shared DTOs, authentication, environment validation, API client, permissions, exceptions, and test utilities.
- `Dockerfile`: Builds the frontend and backend into one production container.
- MongoDB integration through Mongoose.
- S3-compatible media storage through the MinIO client.
- Socket.IO/WebSocket support.
- SMTP email delivery through Nodemailer.
- Optional Mistral AI integration.

The production container serves the built frontend and runs the NestJS backend. Permanent data must be stored in managed external services rather than inside the container filesystem.

## Deployment flowchart

The following Mermaid diagram shows how users, the application container, persistent storage, email, AI, and backups interact. VitePress uses Markdown-It for Markdown rendering, so Mermaid support must be enabled by the documentation site's Mermaid integration or plugin for this block to render as a diagram rather than source code. citeturn0search2turn0search5

```mermaid
flowchart TD
    U[Users] -->|HTTPS| URL[Public application URL]
    URL --> R[Render Web Service]

    subgraph APP[Production Docker container]
        R --> C[Vue frontend]
        R --> API[NestJS API]
        API --> AUTH[Authentication and sessions]
        API --> WS[REST API and WebSockets]
    end

    API -->|Mongoose connection| DB[(MongoDB Atlas)]
    DB --> B[Database backups]
    API -->|S3-compatible API| R2[(Cloudflare R2)]
    R2 --> MEDIA[DPP files and profile pictures]
    API -->|SMTP or HTTPS email API| MAIL[Email provider]
    API -->|Optional API calls| AI[Mistral AI]
    GH[GitHub repository] -->|Push triggers deployment| R

    classDef external fill:#eef6ff,stroke:#2563eb,stroke-width:1px;
    classDef app fill:#ecfdf5,stroke:#059669,stroke-width:1px;
    class U,URL,DB,R2,MAIL,AI,GH,B,MEDIA external;
    class R,C,API,AUTH,WS app;
```

### Flow explanation

1. Users access the public HTTPS URL.
2. Render runs the Docker container containing the Vue frontend and NestJS backend.
3. The backend stores structured data such as users, sessions, organizations, passports, and metadata in MongoDB Atlas.
4. Uploaded DPP files and profile pictures are stored in Cloudflare R2 instead of the temporary application filesystem.
5. The backend sends verification, reset, and notification emails through the configured email provider.
6. Optional AI functionality calls Mistral AI.
7. GitHub remains the source of truth and can trigger a new Render deployment when changes are pushed.
8. MongoDB backups protect the database independently from the application host.

### Text-only fallback

If the documentation site does not have Mermaid rendering enabled, the architecture is:

```text
Users
  |
  v
HTTPS public URL
  |
  v
Render Web Service
  |
  +--> Vue frontend + NestJS backend
  |
  +--> MongoDB Atlas ----> Database backups
  |
  +--> Cloudflare R2 ----> DPP files and profile pictures
  |
  +--> Email provider
  |
  +--> Optional Mistral AI

GitHub repository ----> Render deployment
```

## Component responsibilities

### 1. Render Web Service

**Function:** Runs the production Docker container.

The container builds and starts:

- The Vue client.
- The NestJS API.
- API routes under `/api`.
- Public product-passport routes.
- WebSocket connections.
- Runtime frontend configuration.

The application can sleep on a free service. When it wakes, it reconnects to MongoDB, R2, the email provider, and any configured AI provider. Sleeping does not remove data stored in MongoDB Atlas or R2.

Do not use the Render container filesystem for permanent uploads or database files because hosted container storage is not the durable data store for this architecture.

### 2. MongoDB Atlas

**Function:** Stores structured application data.

MongoDB stores data such as:

- Users and sessions.
- Organizations and memberships.
- Digital product passports.
- Templates and environments.
- Unique product identifiers.
- Permalinks and presentation configurations.
- Media metadata.

Use `OPEN_DPP_MONGODB_URI` to connect the application to Atlas. For an Indian deployment, select an available AWS Mumbai region when the selected Atlas cluster tier supports it.

### 3. Cloudflare R2

**Function:** Stores binary files and media.

R2 stores:

- Uploaded product-passport files.
- Product media.
- Profile pictures.
- Other object-storage content.

Create two buckets:

```text
open-dpp-files
open-dpp-profile-pictures
```

Use an S3-compatible endpoint, port `443`, and SSL enabled for hosted R2:

```env
OPEN_DPP_S3_ENDPOINT=ACCOUNT_ID.r2.cloudflarestorage.com
OPEN_DPP_S3_PORT=443
OPEN_DPP_S3_SSL=true
```

### 4. Email provider

**Function:** Sends verification, password-reset, and application emails.

The local Docker Compose setup uses Mailpit, which is intended for local development. A hosted deployment needs a real email provider.

Configure SMTP or an email API according to the provider supported by the application. Keep the sender address verified.

### 5. Mistral AI

**Function:** Provides optional AI functionality.

The AI provider is not required for the core database, authentication, passport, or media functionality. If enabled, usage is billed separately by the AI provider.

### 6. GitHub

**Function:** Source control and deployment trigger.

GitHub stores the source code and Dockerfile. Render can automatically redeploy when changes are pushed to the configured branch.

## Option comparison

### Option A: Fully free demo deployment

```text
Render Free
MongoDB Atlas Free
Cloudflare R2 Standard free allowance
Email only if compatible with the hosting restrictions
```

#### Estimated cost

| Component | Monthly cost | Notes |
| --- | ---: | --- |
| Render Free | ₹0 | Service may sleep when idle. |
| MongoDB Atlas Free | ₹0 | 512 MB data and index storage. |
| Cloudflare R2 | ₹0 initially | Free allowance for Standard storage and operations. |
| Domain | ₹0 | Use the provider subdomain. |
| Mistral AI | Usage-based | Only if AI features are used. |
| **Total** | **Approximately ₹0** | Subject to free-tier limits. |

#### Advantages

- No hosting bill.
- Good for a college project or proof of concept.
- External MongoDB and R2 keep runtime data separate from the sleeping web service.
- Easy to upgrade later.

#### Disadvantages

- Slow first request after sleeping.
- Free service can be restarted.
- Limited database capacity.
- Limited or no production-grade backups on the free database tier.
- Email delivery may require an HTTPS email API or a paid application host.
- Not suitable for guaranteed availability.

### Option B: Free application with paid database

```text
Render Free
MongoDB Atlas Flex
Cloudflare R2 Standard
Email provider
```

#### Estimated cost

Atlas Flex currently costs between `$8` and `$30` for 30 days of usage, with 5 GB storage, 100 operations/second included in the base price, and a monthly cap. At a planning conversion of `$1 = ₹90`, that is approximately `₹720–₹2,700/month`. The actual invoice can differ.

| Component | Monthly cost | Notes |
| --- | ---: | --- |
| Render Free | ₹0 | Application can sleep. |
| MongoDB Atlas Flex | ₹720–₹2,700 | Usage-based, 5 GB included, backups available. |
| Cloudflare R2 | ₹0 initially | Usage-based after the free allowance. |
| Domain | Optional | Add separately if required. |
| **Total** | **Approximately ₹720–₹2,700** | Before tax and other usage. |

#### Advantages

- Application hosting remains free.
- More database capacity than Atlas Free.
- Backups are available on Flex.
- Suitable when the site can sleep but stored information is important.
- Better long-term safety than relying on the free database tier.

#### Disadvantages

- Database billing continues even when Render is sleeping.
- Atlas Flex is shared infrastructure and is not the same as a dedicated production cluster.
- Requires a payment method and cost monitoring.
- R2, AI, email, and domain usage can add cost.

### Option C: Paid always-on application with free database

```text
Paid Render Web Service
MongoDB Atlas Free
Cloudflare R2 Standard
Email provider
```

#### Estimated cost

Render compute is priced separately from the workspace plan and depends on the selected CPU and memory configuration. Use the Render pricing page and dashboard for the current INR-equivalent estimate. MongoDB Atlas Free is ₹0 but limited to 512 MB and is not recommended for important production data.

#### Advantages

- No application cold start.
- More reliable email and WebSocket behavior than a sleeping free service.
- Keeps database costs at zero initially.
- Useful when the application must remain responsive.

#### Disadvantages

- You pay while the application is idle.
- Atlas Free remains capacity-limited.
- Database backup and durability remain weaker than Atlas Flex.
- Total cost may be higher than paying for Atlas Flex while keeping the app on Render Free.

### Option D: Paid application and paid database

```text
Paid Render Web Service
MongoDB Atlas Flex
Cloudflare R2 Standard
Email provider
```

#### Estimated cost

| Component | Monthly cost | Notes |
| --- | ---: | --- |
| Render paid compute | Provider dashboard price | Depends on CPU and RAM. |
| MongoDB Atlas Flex | ₹720–₹2,700 | Depends on usage, capped at $30/month according to current Atlas Flex pricing. |
| Cloudflare R2 | Usage-based | Free allowance applies first. |
| Email | Free or usage-based | Depends on provider and volume. |
| **Total** | **More than Option B** | Best when responsiveness matters. |

#### Advantages

- No application sleeping.
- Better response time and user experience.
- More appropriate for real users.
- Better database capacity and backups.
- Easier to operate as a small production service.

#### Disadvantages

- Highest recurring cost among these options.
- Requires monitoring multiple usage-based services.
- May be unnecessary if the website receives traffic only occasionally.

## Cost summary in Indian rupees

The following values use a planning conversion of `$1 = ₹90`. They are not a quotation; exchange rates, taxes, and provider billing policies may change.

| Deployment model | Approximate monthly infrastructure cost | Best for |
| --- | ---: | --- |
| Render Free + Atlas Free + R2 free allowance | ₹0 | Demo, learning, college project |
| Render Free + Atlas Flex + R2 | ₹720–₹2,700 | Important data with low traffic |
| Paid Render + Atlas Free + R2 | Provider-dependent | Always-on application with low database usage |
| Paid Render + Atlas Flex + R2 | Paid Render cost + ₹720–₹2,700 | Small production deployment |

Potential additional costs:

- Custom domain registration.
- Mistral API usage.
- Email volume beyond the provider's free allowance.
- R2 storage and operations beyond the free allowance.
- Database usage beyond the Atlas Flex base allowance.
- Taxes and payment-provider conversion charges.

## Most recommended option for this project

### Recommended stack: low-traffic, persistent-data deployment

```text
Application: Render Free Web Service
Database: MongoDB Atlas Flex in an India region when available
Object storage: Cloudflare R2 Standard
Email: Provider-supported email API or SMTP provider
AI: Mistral only when required
Source code: GitHub
Backups: Atlas backup plus separate R2 object backup
```

### Why this is the best balance

1. The application does not need to be open continuously, so Render Free avoids paying for idle application compute.
2. MongoDB Atlas Flex protects the important structured data with more capacity and backup capability than the free database tier.
3. Cloudflare R2 keeps uploaded files outside the ephemeral application container.
4. The services can be upgraded independently.
5. The expected base cost is lower than paying for an always-on application server and a dedicated database.
6. The architecture matches the existing Dockerfile, MongoDB integration, and S3-compatible media service in this repository.

### When to choose the fully free stack instead

Choose Render Free plus Atlas Free if:

- The project is only for learning or demonstration.
- Losing data would not be serious.
- The database will remain below 512 MB.
- You can tolerate slow first requests.
- You are willing to export backups manually.

### When to upgrade the application host

Move from Render Free to a paid web service when:

- Users complain about cold starts.
- WebSocket connections must remain active.
- Email delivery requires outbound SMTP that the free host restricts.
- The application receives regular traffic.
- The service reaches memory or CPU limits.

### When to upgrade the database

Move from Atlas Flex to a dedicated cluster when:

- The database needs guaranteed CPU and memory.
- Query performance becomes unpredictable.
- You need advanced monitoring or higher availability.
- Storage or traffic exceeds Flex limits.
- The application becomes business-critical.

## Minimum production environment variables

```env
OPEN_DPP_PORT=10000
OPEN_DPP_URL=https://your-domain.example

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
OPEN_DPP_MAIL_USER=YOUR_SMTP_USER
OPEN_DPP_MAIL_PASSWORD=YOUR_SMTP_PASSWORD
OPEN_DPP_MAIL_SENDER_ADDRESS=verified@example.com

OPEN_DPP_AUTH_SECRET=GENERATE_A_LONG_RANDOM_SECRET
```

Never commit real values from this example to GitHub. Configure them in the hosting provider's secret or environment-variable settings.

## Operational checklist

- Create MongoDB Atlas and configure a database user.
- Create the two R2 buckets before deploying the application.
- Configure the Render environment variables.
- Configure a verified email sender.
- Test login, registration, passport creation, file upload, and email delivery.
- Configure database backups or exports.
- Back up R2 buckets separately from the Git repository.
- Monitor monthly usage and set billing alerts where available.
- Upgrade Render or Atlas only when observed usage requires it.

## Pricing sources

Check the provider dashboards before purchase because pricing and free-tier conditions can change:

- [Render pricing](https://render.com/pricing)
- [MongoDB Atlas Flex costs](https://www.mongodb.com/docs/atlas/billing/atlas-flex-costs/)
- [MongoDB Atlas cluster comparison](https://www.mongodb.com/docs/atlas/manage-clusters/)
- [Cloudflare R2 pricing](https://developers.cloudflare.com/r2/pricing/)

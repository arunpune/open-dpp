# Open-DPP System Architecture

## Introduction

Open-DPP is an open-source platform for managing Digital Product Passports (DPPs).

A Digital Product Passport stores important information about a product throughout its lifecycle, including:

- Product identity
- Manufacturer information
- Materials and components
- Sustainability data
- Product history
- Repair and recycling information
- Traceability events

Open-DPP provides a web application, backend APIs, database storage, file storage, authentication, permissions, analytics, and AI integrations.

---

## Index

1. [Technology Stack](#technology-stack)
2. [System Architecture](#system-architecture)
3. [Repository Structure](#repository-structure)
4. [Main Components](#main-components)
   - [Frontend Application](#1-frontend-application)
   - [Backend API](#2-backend-api)
   - [Digital Product Passport](#3-digital-product-passport)
   - [Asset Administration Shell](#4-asset-administration-shell)
   - [Authentication and User Management](#5-authentication-and-user-management)
   - [Organizations and Permissions](#6-organizations-and-permissions)
   - [Database and File Storage](#7-database-and-file-storage)
   - [Media Management](#8-media-management)
   - [Templates and Bulk Import](#9-templates-and-bulk-import)
   - [Analytics and Activity History](#10-analytics-and-activity-history)
   - [AI and MCP Integration](#11-ai-and-mcp-integration)
   - [Public Presentations and Permalinks](#12-public-presentations-and-permalinks)
   - [Unique Product Identifiers](#13-unique-product-identifiers)
   - [Shared Packages](#14-shared-packages)
   - [Documentation and Testing](#15-documentation-and-testing)
5. [Request Flow](#request-flow)
6. [How to Run the Project](#how-to-run-the-project)

---

## Technology Stack

| Area | Technology |
|---|---|
| Frontend | Vue 3, TypeScript, Vite |
| Backend | NestJS, TypeScript, Express |
| Database | MongoDB |
| File storage | RustFS, S3-compatible storage |
| Authentication | Better Auth |
| Permissions | CASL and policy guards |
| UI components | PrimeVue and Tailwind CSS |
| State management | Pinia |
| Validation | Zod |
| Build system | pnpm and Turborepo |
| Backend testing | Jest |
| Frontend testing | Vitest |
| End-to-end testing | Playwright |
| Documentation | VitePress |
| AI integration | LangChain, LangGraph, Mistral, MCP |
| Deployment | Docker and Docker Compose |

---

## System Architecture

Open-DPP uses a frontend-backend architecture.

```text
┌──────────────────────────────────────┐
│          Vue Web Application          │
│          apps/client                 │
│                                      │
│  - User interface                    │
│  - Passport management               │
│  - Organization management           │
│  - Authentication screens            │
│  - Public passport presentation      │
└──────────────────┬───────────────────┘
                   │
                   │ REST API, WebSocket, Authentication
                   ▼
┌──────────────────────────────────────┐
│          NestJS Backend API           │
│          apps/main                   │
│                                      │
│  - Business logic                    │
│  - Passport management               │
│  - User management                   │
│  - Permission control                │
│  - File management                   │
│  - Analytics                         │
│  - AI and MCP services               │
└──────────────┬───────────────┬───────┘
               │               │
               ▼               ▼
┌────────────────────┐  ┌────────────────────┐
│      MongoDB        │  │       RustFS        │
│                     │  │                    │
│  Application and    │  │  Uploaded files     │
│  passport data      │  │  and media files    │
└────────────────────┘  └────────────────────┘
               │
               ▼
┌────────────────────┐
│    Mailpit / SMTP  │
│                    │
│  Application email │
└────────────────────┘
```

### Backend Layers

The backend is divided into four main layers:

```text
Presentation Layer
    Controllers, routes, decorators, and request validation

Application Layer
    Services and business use cases

Domain Layer
    Product passport models and business rules

Infrastructure Layer
    MongoDB repositories, file storage, email, and external services
```

This separation makes the system easier to understand, test, and maintain.

---

## Repository Structure

```text
open-dpp/
│
├── apps/
│   ├── client/                    Vue frontend application
│   ├── main/                      NestJS backend application
│   └── e2e/                       Playwright end-to-end tests
│
├── packages/
│   ├── api-client/                Shared typed API client
│   ├── dto/                       Shared data types and validation schemas
│   ├── env/                       Environment configuration and validation
│   ├── exception/                 Error handling and exception filters
│   ├── permission/                Permission and authorization helpers
│   └── testing/                   Shared testing utilities
│
├── docs/                          VitePress documentation
├── docker/                        Docker support files
├── scripts/                       Setup and code-generation scripts
│
├── docker-compose.yml             Example deployment configuration
├── docker-compose.dev.yml         Development services
├── Makefile                       Common development commands
├── package.json                   Root project scripts
├── pnpm-workspace.yaml            Workspace configuration
└── turbo.json                     Turborepo task configuration
```

---

# Main Components

## 1. Frontend Application

**Location:** `apps/client`

The frontend is the web application used by people who manage product passports.

### Main functionalities

- User registration and login
- Password reset
- Organization selection
- Product passport creation
- Product passport editing
- AAS and submodel management
- Template management
- Media upload
- Public passport viewing
- Analytics display
- Administration
- Language selection

### Main technologies

- Vue 3
- Vue Router
- Pinia
- PrimeVue
- Tailwind CSS
- Vue I18n
- Better Auth client
- Shared API client

The frontend uses Pinia stores to manage:

- User information
- Current organization
- Selected passport
- Language
- Layout state
- Application settings

### Frontend layouts

```text
Main Layout
    Used for the normal authenticated application

Presentation Layout
    Used for public passport pages

None Layout
    Used for login, registration, and password reset pages
```

---

## 2. Backend API

**Location:** `apps/main`

The backend is the central server of the platform.

### Main functionalities

- REST API
- Authentication
- Organization management
- Passport management
- AAS management
- Media management
- Analytics
- WebSocket communication
- AI services
- MCP services
- OpenAPI documentation

The backend is divided into feature modules, including:

```text
AasModule
PassportsModule
AuthModule
OrganizationsModule
UsersModule
MediaModule
AnalyticsModule
TemplatesModule
PolicyModule
EmailModule
AiModule
```

The backend also provides:

- API versioning
- Request validation
- CORS support
- Error handling
- Logging
- Correlation IDs
- Request size limits
- Swagger/OpenAPI documentation

---

## 3. Digital Product Passport

**Locations:**

- `apps/main/src/passports`
- `apps/main/src/digital-product-document`

The Digital Product Passport is the main business feature of Open-DPP.

A passport stores information about a product during its complete lifecycle.

### Main functionalities

- Create a passport
- View a passport
- Update a passport
- Delete a passport
- Change passport status
- Create a passport from a template
- Import a passport
- Export a passport
- Generate a product identifier
- View passport activity
- Download passport activity
- Create public passport links

### Example API operations

```text
GET     /passports
GET     /passports/:id
POST    /passports
DELETE  /passports/:id
PUT     /passports/:id/status
GET     /passports/:id/export
POST    /passports/import
```

Passports are connected to organizations. Users can only access passports that they are allowed to access.

---

## 4. Asset Administration Shell

**Location:** `apps/main/src/aas`

An Asset Administration Shell, also called an AAS, is a structured digital representation of a physical product or asset.

The structure is:

```text
Digital Product Passport
└── Environment
    ├── Asset Administration Shells
    └── Submodels
        └── Submodel Elements
```

### Main functionalities

- Create AAS structures
- Read AAS structures
- Update AAS information
- Create submodels
- Update submodels
- Delete submodels
- Read submodel values
- Create submodel elements
- Update submodel elements
- Delete submodel elements
- Move submodel elements
- Add rows and columns
- Remove rows and columns
- Reorder columns
- Create groups from columns
- Move columns into groups

Nested elements are accessed using an `idShortPath`.

---

## 5. Authentication and User Management

**Location:** `apps/main/src/identity`

This component manages users and login-related operations.

### Main functionalities

- User registration
- User login
- User logout
- Password reset
- Email verification
- Email address changes
- User profile management
- User roles
- API keys
- OAuth providers
- Administrator access

The frontend uses Better Auth to communicate with the backend authentication system.

The backend uses an authentication guard:

```text
Incoming request
       │
       ▼
AuthGuard
       │
       ├── Authenticated user → Continue
       └── Unauthenticated user → Reject request
```

---

## 6. Organizations and Permissions

**Locations:**

- `apps/main/src/identity/organizations`
- `apps/main/src/policy`
- `packages/permission`

Open-DPP supports multiple organizations.

Each organization can have its own:

- Users
- Members
- Roles
- Product passports
- Templates
- Settings
- Permissions

### Example permissions

- A user can create an organization.
- An organization member can view the organization.
- An organization owner can update the organization.
- An organization owner can delete the organization.
- A member can access allowed passports.
- A user cannot access another organization's private data.

### Permission flow

```text
AuthGuard
    Checks whether the user is logged in

PolicyGuard
    Checks whether the user is allowed to perform the operation

Organization information
    Identifies the current organization

Member role
    Identifies the user's role in the organization
```

---

## 7. Database and File Storage

### MongoDB

MongoDB is the main application database.

It stores:

- Users
- Organizations
- Passports
- AAS data
- Submodels
- Templates
- Policies
- Activity history
- Analytics
- API keys
- Settings
- Traceability events

The backend uses Mongoose and MongoDB transactions.

### RustFS

RustFS is an S3-compatible object storage system.

It stores:

- Product images
- Profile pictures
- Uploaded passport files
- Other media files

The Docker setup creates separate storage buckets for:

```text
Application files
Profile pictures
```

### Mailpit

Mailpit is used during development to test email features.

It can display:

- Password reset emails
- Organization invitations
- Email verification emails
- Other application emails

---

## 8. Media Management

**Location:** `apps/main/src/media`

The media module manages files uploaded by users.

### Main functionalities

- Upload files
- Store files in RustFS
- Download files
- Detect file types
- Process images
- Store profile pictures
- Upload passport files
- Optional virus scanning

ClamAV can be configured for virus scanning. If it is not configured, uploaded files are not scanned.

---

## 9. Templates and Bulk Import

### Templates

**Location:** `apps/main/src/templates`

Templates are reusable passport structures.

They allow users to create new passports using a predefined format.

```text
Create template
      │
      ▼
Select template
      │
      ▼
Create passport
      │
      ▼
Add product information
```

### Bulk Import

**Location:** `apps/main/src/bulk-import`

Bulk import allows users to import many products or passports.

It supports:

- Import configuration
- Import runs
- Data parsing
- Item-level status
- Import results
- Error tracking

---

## 10. Analytics and Activity History

**Locations:**

- `apps/main/src/analytics`
- `apps/main/src/activity-history`
- `apps/main/src/traceability-events`

These components record how passports are used and changed.

### Analytics features

- Passport page views
- Usage statistics
- Time-based reports
- Passport metrics
- Page-view tracking

### Activity history features

- Record passport changes
- Record user actions
- Store timestamps
- Store activity types
- Filter activities by date
- Filter activities by product path
- Download activity history

This gives users an audit history of important product passport changes.

---

## 11. AI and MCP Integration

**Locations:**

- `apps/main/src/ai`
- `apps/main/src/mcp`

Open-DPP includes AI and Model Context Protocol functionality.

### Technologies

- LangChain
- LangGraph
- Mistral
- Model Context Protocol
- MCP adapters

### Main functionalities

- AI configuration
- AI chat
- MCP client connection
- MCP server support
- Agent-server API
- AI model configuration
- External AI tool communication

The backend connects to the MCP client during startup.

---

## 12. Public Presentations and Permalinks

**Locations:**

- `apps/main/src/presentation-configurations`
- `apps/main/src/permalink`
- `apps/client/src/view`
- `apps/client/src/layout/Presentation.vue`

This component controls how passport information is shown to external users.

### Main functionalities

- Create presentation configurations
- Select which passport information is visible
- Create public passport pages
- Generate stable passport URLs
- Display passports without requiring the management dashboard
- Support different presentation layouts

The public presentation view is separate from the authenticated application.

---

## 13. Unique Product Identifiers

**Locations:**

- `apps/main/src/unique-product-identifier`
- `packages/dto/src/unique-product-identifiers`

The system supports different product identification methods:

- Internal product identifiers
- GS1 identifiers
- GTIN
- Batch numbers
- Serial numbers
- GS1 Digital Link formats

### Example resolver paths

```text
/01/:gtin
/01/:gtin/10/:batch
/01/:gtin/21/:serial
```

These identifiers can be used to find a product passport.

---

## 14. Shared Packages

### `packages/api-client`

Provides a reusable typed API client.

It supports:

- DPP operations
- AAS operations
- Organizations
- Users
- Templates
- Policies
- Permalinks
- Analytics
- Media
- Status checks
- Agent-server communication

The main class is:

```text
OpenDppClient
```

### `packages/dto`

Contains shared data types and validation schemas.

It defines models for:

- Passports
- AAS
- Submodels
- Users
- Organizations
- Policies
- Templates
- Analytics
- Media
- Activity history
- Product identifiers
- API versions

The frontend and backend use the same DTO definitions.

### `packages/env`

Validates environment variables using Zod.

It checks configuration for:

- MongoDB
- RustFS
- SMTP
- Authentication
- OAuth
- AI
- ClamAV
- Application settings

### `packages/exception`

Provides common error handling.

It contains:

- Domain errors
- Service errors
- Validation helpers
- HTTP exception filters
- WebSocket exception filters
- Not-found handling
- Permission error handling

### `packages/permission`

Contains permission rules using CASL.

It helps determine whether a user can:

- Create a resource
- Read a resource
- Update a resource
- Delete a resource

### `packages/testing`

Contains shared testing utilities and test factories.

---

## 15. Documentation and Testing

### Documentation

**Location:** `docs`

The documentation website uses VitePress.

It contains:

- Getting-started guides
- Configuration information
- Reference documentation
- Usage guides
- Changelog
- API documentation

The backend can generate OpenAPI documentation.

### Testing

The repository uses multiple testing tools:

```text
Jest
    Backend unit and integration tests

Vitest
    Frontend tests

Playwright
    Browser-based end-to-end tests

MongoDB Memory Server
    Temporary MongoDB database for tests
```

End-to-end tests are located in:

```text
apps/e2e
```

---

# Request Flow

A normal request follows this process:

```text
1. The user opens the Vue web application
2. The frontend checks the user's login session
3. The user selects an organization
4. The frontend calls the shared API client
5. The request reaches the NestJS backend
6. AuthGuard checks the user's login
7. PolicyGuard checks the user's permissions
8. Request data is validated
9. The controller calls an application service
10. The service applies business rules
11. Data is read from or written to MongoDB
12. Files are stored in RustFS when needed
13. Activity history is recorded
14. The backend returns a response
15. The frontend updates the screen
```

## Example: Creating a Passport

```text
User enters passport information
          │
          ▼
Vue frontend sends the request
          │
          ▼
PassportController receives the request
          │
          ▼
AuthGuard and PolicyGuard check access
          │
          ▼
PassportService creates the passport
          │
          ▼
MongoDB stores the passport
          │
          ▼
Unique product identifier is created
          │
          ▼
Response is returned to the frontend
```

---

# How to Run the Project

## Install dependencies

```bash
pnpm install
```

## Configure environment variables

Create a development environment file:

```bash
cp .env.dev.example .env.dev
```

Configure values for:

- MongoDB
- RustFS
- SMTP
- Authentication
- Application URL
- Application port

## Start development services

```bash
make dev
```

This starts services such as:

- MongoDB
- RustFS
- Mailpit

## Start the applications

```bash
pnpm run dev
```

## Start only the backend

```bash
pnpm run dev:main
```

## Build the project

```bash
pnpm run build
```

## Build only the backend

```bash
pnpm run build:main
```

## Build the documentation

```bash
pnpm run build:docs
```

## Run all tests

```bash
pnpm run test
```

## Run backend tests

```bash
pnpm run test:main
```

## Run frontend tests

```bash
pnpm run test:client
```

## Run end-to-end tests

```bash
pnpm run test:e2e
```

---

# Summary

Open-DPP is a complete platform for creating and managing Digital Product Passports.

Its major parts are:

```text
Vue frontend
    Provides the user interface

NestJS backend
    Provides APIs and business logic

MongoDB
    Stores application and passport data

RustFS
    Stores uploaded files and media

Better Auth
    Handles users and authentication

CASL and policy guards
    Control permissions

Shared DTO package
    Keeps frontend and backend data consistent

API client
    Provides reusable API communication

AI and MCP modules
    Enable AI-powered integrations

Documentation and testing tools
    Support development and maintenance
```

# New Business Launch Radar

An internal business-discovery and lead-intelligence platform for finding recently registered, licensed, opened, or expanded businesses from permitted public sources.

The platform converts fragmented government records, licensing feeds, public announcements, and carefully controlled enrichment lookups into structured, evidence-backed business opportunities. Every record retains its source, event type, date confidence, and review status from discovery through CRM delivery.

> A launch signal is not buying intent. Registration, licensing, opening, digital opportunity, and willingness to buy are separate signals and must remain separate throughout the system.

## Status

This repository is intended to implement the recommended **MVP-B: Registration + Licensing Radar**.

Before full development, run the prescribed **14-day Texas validation experiment** to confirm that the available records are commercially useful, sufficiently current, and economical to review.

### Recommended sequence

1. Validate Texas data for 14 days.
2. Build the registration-and-licensing MVP.
3. Add public launch announcements.
4. Add website audits and digital-opportunity classification.
5. Correlate multiple sources into a broader intelligence product.

## Business problem

Flex The Brand Studio currently depends on rented acquisition channels such as paid advertising, lead marketplaces, and directories. These channels are increasingly expensive, competitive, and inconsistent.

The agency needs an owned discovery pipeline that can:

- Identify recently registered, licensed, opened, or expanded businesses.
- Preserve evidence showing why a business was discovered.
- Distinguish legal registration from an actual opening.
- Identify potential web, branding, SEO, e-commerce, or automation gaps.
- Remove duplicate businesses across multiple sources.
- Prioritize opportunities without presenting assumptions as facts.
- Require human verification before CRM creation.
- Measure cost, quality, review effort, outreach, and conversion.

## Proposed solution

New Business Launch Radar is a narrow internal intelligence platform, not a nationwide social-media scraper.

The system:

1. Runs scheduled discovery jobs against enabled and permitted sources.
2. Stores raw source responses with provenance and retention metadata.
3. Validates and normalizes business records.
4. Resolves duplicate identities across sources.
5. Extracts typed launch events.
6. Assigns evidence and date confidence.
7. Optionally enriches records through controlled live lookups.
8. Uses deterministic rules and constrained AI classification.
9. Sends uncertain records to a human review queue.
10. Exports only approved leads to the CRM.
11. Records CRM synchronization, suppression, and outreach outcomes.

## What counts as a launch event?

The product deliberately separates different events.

| Event | Example source | Reliability |
|---|---|---|
| Entity registration | Secretary of State or tax registration | Reliable formation date, not an opening date |
| License or permit | City, county, or state open-data portal | Stronger opening proxy |
| Public launch announcement | News, PR, official public announcement | Strong when the date and business identity are clear |
| New location or expansion | Licensing, announcement, or official record | Moderate to strong with corroboration |
| New directory listing | Google, Yelp, or niche directory | Weak; first-seen date is not a launch date |
| Existing business with a new listing | Directory-only observation | False-positive class to detect |

The following dates must never be collapsed into one field:

- `event_date`
- `discovery_date`
- `publication_date`
- `registration_date`
- `opening_date`
- `listing_date`
- `updated_at`

## MVP scope

### Included

- Secure internal user access
- Configurable search jobs
- Texas registration/taxpayer adapter
- One or more municipal licensing or permit adapters
- Adapter framework for additional states and jurisdictions
- Raw source storage
- Validation and schema enforcement
- Business and address normalization
- Entity resolution and duplicate review
- Typed launch events
- Event evidence and date confidence
- Human review queue
- Basic business and event search
- Lead qualification status
- Assignment and notes
- HubSpot or Airtable export
- CRM synchronization logs and retries
- Suppression and do-not-contact handling
- Audit trail
- API-usage and processing-cost metering
- Monitoring, backups, and deployment documentation

### Optional MVP extensions

- Florida Sunbiz daily SFTP adapter
- New York Socrata adapter
- Brave Search and GDELT announcement evidence
- Public website audit
- PageSpeed Insights
- AI-assisted extraction from public announcements
- Batch review interface
- CSV import and export
- Slack or email job notifications

### Explicit non-goals

- Nationwide scraping of all 50 state websites
- LinkedIn scraping
- Automated Meta, TikTok, or X discovery
- CAPTCHA or anti-bot bypassing
- Private-profile access
- Treating directory listings as verified launch dates
- Inferring owner names or contacts that were never published
- Presenting launch activity as buying intent
- Automated cold outreach
- Autonomous CRM creation without review
- Building a replacement CRM
- Storing third-party data in breach of provider terms

## Architecture

```mermaid
flowchart TD
    SRC["Official Feeds, Permits, News"] --> ADP["Source Adapter Framework"]
    ADP --> RAW["Raw Document Store"]
    RAW --> NORM["Validate and Normalize"]
    NORM --> RES["Entity Resolution"]
    RES --> EVT["Launch Event Detection"]
    EVT --> ENR["Controlled Enrichment"]
    ENR --> CLS["Rules and AI Classification"]
    CLS --> REV["Human Review Queue"]
    REV --> CRM["HubSpot or Airtable"]
    CRM --> REP["Reporting and Outcomes"]
```

Provenance, confidence, retention rules, and human-review status travel with every record through the entire pipeline.

### Architectural approach

Start with a modular monolith:

- One FastAPI application
- One PostgreSQL database
- Redis-backed background jobs
- Separately scalable worker processes
- One web frontend
- Source-specific adapters behind a common contract

Do not begin with microservices or Kubernetes. Service extraction is justified only after volume, team structure, or independent scaling requires it.

## Recommended technology stack

| Layer | Technology | Responsibility |
|---|---|---|
| Frontend | Next.js, React, TypeScript | Internal search, review, lead, and reporting interface |
| UI | Tailwind CSS and shadcn/ui | Responsive, accessible design system |
| Backend | Python and FastAPI | APIs, orchestration, validation, and business logic |
| Schema validation | Pydantic | Strict external and internal data contracts |
| ORM and migrations | SQLAlchemy and Alembic | Persistence and versioned database changes |
| Database | PostgreSQL | System of record and JSONB raw metadata |
| Matching | PostgreSQL `pg_trgm`, RapidFuzz, libpostal | Candidate matching and normalization |
| Queue | Redis with RQ initially | Scheduled and asynchronous processing |
| Production workers | Celery when justified | Higher-volume queues and worker routing |
| Static web retrieval | HTTPX and BeautifulSoup/lxml | Permitted website checks |
| Browser fallback | Playwright in an isolated worker | JavaScript sites only when justified and permitted |
| AI | Low-cost model for extraction; stronger fallback model | Evidence-bound classification, never authoritative facts |
| Local AI option | Ollama | Development, testing, and low-risk bulk extraction |
| Object storage | S3-compatible storage | Raw documents and approved artifacts |
| CRM | HubSpot or Airtable | Approved lead delivery |
| Monitoring | Sentry and infrastructure metrics | Errors, traces, alerts, and worker health |
| Analytics | PostHog or internal metrics | Workflow, review, and product usage |
| Packaging | Docker | Repeatable local and production environments |
| CI/CD | GitHub Actions | Tests, linting, security checks, and deploy gates |
| Hosting | VPS plus managed PostgreSQL | Cost-controlled internal MVP deployment |

## Repository structure

```text
new-business-launch-radar/
├── apps/
│   ├── web/                         # Next.js internal application
│   └── api/                         # FastAPI application
├── radar/
│   ├── adapters/                    # One module per external source
│   ├── ingestion/                   # Scheduling, raw storage, validation
│   ├── normalization/               # Names, addresses, phones, domains
│   ├── resolution/                  # Candidate generation and matching
│   ├── events/                      # Launch-event extraction and evidence
│   ├── enrichment/                  # Live lookup and web audit
│   ├── intelligence/                # Classification and scoring
│   ├── review/                      # Human decisions and corrections
│   ├── crm/                         # CRM adapters and synchronization
│   ├── compliance/                  # Retention, suppression, deletion
│   └── observability/               # Metrics, logs, and cost metering
├── workers/                         # Background worker entrypoints
├── migrations/                      # Alembic migrations
├── specs/                           # Feature specifications
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── fixtures/
│   ├── data_quality/
│   └── e2e/
├── infrastructure/                  # Docker and deployment configuration
├── docs/                            # Architecture and operational guides
├── docker-compose.yml
└── README.md
```

## Source adapter contract

Each source must implement a common lifecycle. Source-specific authentication, pagination, rate limits, field names, retention restrictions, and errors remain isolated inside the adapter.

```python
from typing import Protocol

class SourceAdapter(Protocol):
    name: str
    source_kind: str

    async def discover(self, config): ...
    async def fetch(self, reference): ...
    def validate(self, raw_document): ...
    def normalize(self, raw_document): ...
    def emit_events(self, normalized_record): ...
    def deduplication_keys(self, normalized_record): ...
    def rate_limit_policy(self): ...
    def retention_policy(self): ...
```

Every adapter requires:

- Source owner and authoritative URL
- Access method and credentials
- Commercial-use determination
- Storage and caching rules
- Rate-limit policy
- Retry and backoff policy
- Pagination implementation
- Schema version
- Recorded test fixtures
- Provenance mapping
- Monitoring and disable switch

## Recommended data sources

### Primary discovery

| Source | Role | MVP priority |
|---|---|---:|
| Texas Comptroller permitted files | Registration/taxpayer discovery | Required for validation |
| Municipal license and permit portals | Opening proxy | Required: select 1-2 jurisdictions |
| Florida Sunbiz daily SFTP | Registration discovery | Next source |
| New York Socrata API | Registration discovery | Next source |
| Brave Search and GDELT | Public announcement evidence | Phase 3 |

### Controlled enrichment

| Source | Role | Restriction |
|---|---|---|
| Google Places | Live business lookup | Store only permitted fields and respect caching terms |
| Yelp Fusion | Category and business enrichment | API only; restricted storage; not a discovery feed |
| PageSpeed Insights | Website performance signals | Deterministic audit input |
| RDAP | Domain registration metadata | Do not equate domain date with business launch |
| OpenCorporates | KYB verification | Use under the selected commercial licence |

### Sources to avoid

- LinkedIn automated discovery or scraping
- X full-archive search for the MVP
- Meta or TikTok discovery scraping
- CAPTCHA-gated state portals
- State interfaces that prohibit automation
- Directory-page scraping when an official API or feed is required

## Data model

The database uses layered storage so raw evidence remains separate from normalized entities and sales workflow.

### Core layers

| Layer | Primary tables |
|---|---|
| Raw | `raw_documents`, `ingestion_runs` |
| Normalized | `discovered_records`, `normalized_addresses` |
| Entity | `businesses`, `business_locations`, `business_aliases`, `contacts` |
| Events | `launch_events`, `event_evidence` |
| Intelligence | `enrichments`, `website_audits`, `opportunity_signals`, `lead_scores` |
| Review | `review_tasks`, `review_decisions` |
| CRM | `crm_leads`, `crm_sync_logs`, `outreach_events` |
| Compliance | `suppression_entries`, `deletion_requests`, `audit_logs` |

### Entity relationship overview

```mermaid
erDiagram
    BUSINESSES ||--o{ BUSINESS_LOCATIONS : has
    BUSINESSES ||--o{ CONTACTS : has
    BUSINESSES ||--o{ LAUNCH_EVENTS : generates
    LAUNCH_EVENTS ||--o{ EVENT_EVIDENCE : supported_by
    BUSINESSES ||--o{ WEBSITE_AUDITS : receives
    BUSINESSES ||--o{ LEAD_SCORES : receives
    BUSINESSES ||--o| CRM_LEADS : may_become
```

### Provenance requirements

Every discovered record and event should retain:

- `source_id`
- `source_record_id`
- `source_url`
- `retrieved_at`
- `published_at`
- `raw_document_id`
- `extracted_fields`
- `evidence_text`
- `evidence_confidence`
- `schema_version`
- `processed_at`

## Entity resolution

Never merge businesses using name similarity alone.

Candidate comparison may use:

- Normalized business name
- Registration number
- Normalized address and ZIP code
- Public phone in E.164 format
- Domain
- Coordinates
- City and state
- Public social link

Resolution stages:

1. Normalize
2. Block candidates
3. Compare corroborating fields
4. Calculate transparent match signals
5. Auto-merge only at high confidence
6. Send medium-confidence matches to review
7. Keep low-confidence records separate

Useful libraries include `libpostal`, `RapidFuzz`, `phonenumbers`, and optionally `Splink` after the initial rule-based matcher is validated.

## Confidence model

Keep three components visible:

1. **Launch signal** - Is there reliable evidence of a registration, licence, permit, announcement, or expansion?
2. **Digital-opportunity signal** - Does a deterministic website audit or evidence-backed review indicate a relevant service gap?
3. **Buying-intent signal** - Is there explicit public evidence that the business is looking to purchase services?

Do not collapse them into a misleading black-box score. A priority score may rank the review queue, but reviewers must see its individual components and evidence.

### Verified-event threshold

A launch event is verified only when:

- Business identity is supported by a reliable source.
- Event type is explicit.
- Event evidence is stored.
- Date type and confidence are known.
- Source URL is stored.
- Duplicate review passes.
- A human reviewer confirms the record.

Anything missing remains a candidate.

## AI usage and guardrails

AI is optional and subordinate to deterministic processing.

### Good AI tasks

- Extract an event from public narrative text.
- Classify an industry from public descriptions.
- Distinguish an announcement from unrelated news.
- Summarize stored evidence.
- Suggest a service opportunity from deterministic audit findings.

### Deterministic tasks

- Structured dates
- Registration identifiers
- Phone and address normalization
- JSON-LD parsing
- Exact source-field mapping
- Duplicate constraints
- Suppression checks
- CRM approval gate

### Mandatory AI output fields

```json
{
  "event_type": "grand_opening_announcement",
  "event_date": "2026-09-10",
  "date_confidence": 0.91,
  "industry": "restaurant",
  "evidence_text": "...",
  "evidence_url": "https://example.com/announcement",
  "confidence": 0.88,
  "model": "provider-model-version",
  "prompt_version": "event-extraction-v1",
  "requires_human_review": true
}
```

AI must never invent:

- Opening dates
- Owners
- Contact details
- Budgets
- Buying intent
- Missing evidence

Use a low-cost or local model for bulk extraction and escalate only uncertain cases to a stronger model. Keep the model provider behind an interface so it can be replaced without changing the pipeline.

## Website audit

The MVP audit should begin with inexpensive deterministic checks:

- Site reachability
- HTTP status and redirects
- HTTPS
- Title and meta description
- Mobile viewport
- Contact links
- Social links
- robots.txt and sitemap
- Broken internal links within the approved crawl limit
- PageSpeed Insights

Use evidence-based wording such as:

> The audit found no visible appointment or booking link on the inspected pages.

Avoid unsupported conclusions such as:

> This company needs a new website.

Because the platform fetches user-selected or discovered URLs, server-side request forgery protection is mandatory. Block private and metadata IP ranges, restrict schemes and ports, limit redirects and response sizes, and isolate Playwright when browser rendering is introduced.

## CRM integration

Use an existing CRM rather than building one.

Recommended order:

1. HubSpot for a conventional CRM workflow
2. Airtable for a faster and more flexible internal MVP
3. Google Sheets only for the validation experiment

Only human-approved records may be synchronized.

CRM synchronization must support:

- Create or update
- Duplicate lookup
- Field mapping
- Assigned owner
- Event and evidence links
- Source provenance
- Conflict handling
- Idempotency
- Retry with backoff
- Per-record synchronization log
- Suppression enforcement

## API surface

The exact routes should be finalized in feature specifications. A reasonable MVP surface is:

| Method | Route | Purpose |
|---|---|---|
| `POST` | `/api/v1/search-jobs` | Create a discovery job |
| `GET` | `/api/v1/search-jobs` | List jobs |
| `GET` | `/api/v1/search-jobs/{id}` | Job status and metrics |
| `POST` | `/api/v1/search-jobs/{id}/retry` | Retry failed work |
| `GET` | `/api/v1/businesses` | Search normalized businesses |
| `GET` | `/api/v1/businesses/{id}` | Business, events, evidence, and audit |
| `GET` | `/api/v1/events` | Filter launch events |
| `GET` | `/api/v1/review-tasks` | Human review queue |
| `POST` | `/api/v1/review-tasks/{id}/decision` | Approve, reject, duplicate, or enrich |
| `POST` | `/api/v1/businesses/{id}/audit` | Run a website audit |
| `POST` | `/api/v1/businesses/{id}/crm-sync` | Sync an approved lead |
| `GET` | `/api/v1/reports/quality` | Data-quality and pipeline metrics |
| `GET` | `/api/v1/reports/costs` | Source, API, and AI usage |

## Roles

| Role | Access |
|---|---|
| Administrator | Users, roles, sources, credentials metadata, retention, and all records |
| Technical administrator | Adapters, jobs, logs, health, and deployment operations |
| Reviewer | Review queue, evidence, correction, approval, and rejection |
| Sales representative | Approved leads, assignment, outreach status, and notes |
| CRM manager | CRM mapping, synchronization, duplicates, and reporting |

## Local development

### Prerequisites

- Git
- Docker and Docker Compose
- Node.js 22 or the version pinned in the repository
- Python 3.12 or the version pinned in the repository
- `uv` for Python dependency management
- `pnpm` for frontend dependency management

### Clone and configure

```bash
git clone <repository-url>
cd new-business-launch-radar
cp .env.example .env
```

### Start infrastructure

```bash
docker compose up -d postgres redis
```

### Backend

```bash
cd apps/api
uv sync
uv run alembic upgrade head
uv run uvicorn app.main:app --reload
```

### Frontend

```bash
cd apps/web
pnpm install
pnpm dev
```

### Worker

```bash
cd apps/api
uv run rq worker radar
```

The exact commands should be updated if the implementation chooses a different monorepo runner or queue package.

## Environment variables

Create `.env.example` with names only. Never commit real credentials.

```dotenv
# Application
APP_ENV=development
APP_BASE_URL=http://localhost:3000
API_BASE_URL=http://localhost:8000
SECRET_KEY=

# Database and queue
DATABASE_URL=postgresql+psycopg://radar:radar@localhost:5432/radar
REDIS_URL=redis://localhost:6379/0

# Authentication
AUTH_ISSUER=
AUTH_AUDIENCE=
AUTH_CLIENT_ID=
AUTH_CLIENT_SECRET=

# Discovery sources
TX_SOURCE_PATH=
FL_SFTP_HOST=
FL_SFTP_USERNAME=
FL_SFTP_PASSWORD=
NY_SOCRATA_APP_TOKEN=

# Search and enrichment
BRAVE_SEARCH_API_KEY=
GOOGLE_PLACES_API_KEY=
GOOGLE_PAGESPEED_API_KEY=
YELP_API_KEY=
OPENCORPORATES_API_KEY=

# AI
AI_PROVIDER=
AI_MODEL=
AI_API_KEY=
AI_MONTHLY_BUDGET_USD=

# CRM
CRM_PROVIDER=
HUBSPOT_ACCESS_TOKEN=
AIRTABLE_ACCESS_TOKEN=
AIRTABLE_BASE_ID=

# Storage and monitoring
S3_ENDPOINT=
S3_BUCKET=
S3_ACCESS_KEY_ID=
S3_SECRET_ACCESS_KEY=
SENTRY_DSN=

# Notifications
SLACK_WEBHOOK_URL=
EMAIL_API_KEY=
```

## Specification-driven development

Every feature should begin with a specification in `specs/`.

Each specification should define:

- Business objective
- User story
- Scope and non-goals
- Source and licensing assumptions
- Data contracts
- Database changes
- API contract
- Permissions
- Retry and failure behaviour
- Retention requirements
- Observability
- Acceptance criteria
- Unit, integration, fixture, data-quality, and end-to-end tests
- Rollback notes

### Recommended specification order

1. Authentication and roles
2. Source registry and adapter interface
3. Raw document ingestion
4. Texas source adapter
5. Normalization
6. Entity resolution
7. Launch-event taxonomy
8. Evidence and confidence
9. Review workflow
10. CRM export
11. Suppression and deletion
12. Monitoring and cost metering
13. Announcement sources
14. Website audit
15. AI extraction and classification

### Claude CLI workflow

1. Ask Claude CLI to inspect the repository and selected specification.
2. Require a technical plan before code changes.
3. Confirm assumptions and external-source restrictions.
4. Implement one bounded feature.
5. Add recorded fixtures and tests.
6. Run formatting, linting, types, tests, and migrations.
7. Review security-sensitive and licensing-sensitive changes manually.
8. Commit in small reversible units.
9. Document new environment variables and operational changes.
10. Close the feature only with acceptance evidence.

Claude CLI can accelerate code, fixtures, tests, schemas, migrations, and documentation. Humans remain responsible for data licences, API accounts, security sign-off, production deployment, data-quality acceptance, and sales workflow.

## Quality checks

Typical commands:

```bash
# Backend
uv run ruff check .
uv run ruff format --check .
uv run mypy .
uv run pytest

# Frontend
pnpm lint
pnpm typecheck
pnpm test
pnpm test:e2e
pnpm build
```

## Testing strategy

### Unit tests

- Source field mapping
- Normalization
- Phone and address handling
- Deduplication keys
- Match signals
- Event classification rules
- Confidence calculation
- Scoring components
- Retention rules

### Adapter integration tests

- Recorded licensed fixtures
- Pagination
- Rate limits
- Malformed responses
- Empty responses
- Changed schemas
- Authentication failures
- Retry behaviour

Do not run uncontrolled live-source tests in ordinary CI.

### End-to-end tests

- Discover record
- Store raw source
- Normalize business
- Detect or review duplicate
- Create event and evidence
- Review event
- Approve lead
- Synchronize to test CRM
- Verify audit trail

### Data-quality gates

Deployment acceptance depends on more than passing software tests. Track:

- Unique record rate
- Duplicate rate
- False-positive event rate
- Missing required data
- Event-date confidence
- Enrichment success
- Review time per record
- Cost per approved lead

## 14-day Texas validation experiment

Before building all phases, run a controlled validation.

### Configuration

- **State:** Texas
- **Industries:** Restaurants and home services
- **Primary source:** Texas Comptroller permitted new-taxpayer data
- **Evidence sources:** Brave Search and/or GDELT announcements
- **Duration:** 14 days

### Measure

- Records discovered
- Unique businesses after deduplication
- Valid launch events
- False positives
- Duplicate rate
- Missing-data percentage
- Enrichment success
- Human verification time per record
- API cost
- Qualified opportunities
- CRM-ready leads

### Go criteria

- Useful unique and verified businesses are found each week.
- False positives are approximately below the agreed threshold, initially targeted at 20%.
- Deduplication performs acceptably.
- Review time is operationally manageable.
- Cost per qualified lead is commercially reasonable.

### No-go criteria

- Launch dates cannot be represented honestly.
- Most records are old businesses or duplicates.
- Human review overwhelms the sales team.
- Cost plus review time exceeds the expected lead value.
- Source terms or licensing prevent the intended use.

If validation fails, license a suitable data provider rather than building a larger scraper.

## Delivery roadmap

| Phase | Deliverable | Estimated effort |
|---|---|---:|
| 0 | 14-day source-validation experiment | 14 calendar days |
| 1 | Event model, PostgreSQL schema, one adapter, basic deduplication, admin UI | 80-140 hours |
| 2 | Additional adapters, stronger entity resolution, provenance | 120-200 hours |
| 3 | Brave/GDELT, AI extraction, human review queue | 120-200 hours |
| 4 | Website audit, PageSpeed, three-signal scoring, SSRF controls | 100-160 hours |
| 5 | CRM sync, suppression, approval gate, audit logs | 80-140 hours |
| 6 | Additional states, sources, and verticals | Ongoing |

Phases 1-5 total approximately **500-840 engineering hours**, plus security, legal, data-quality, and operational setup.

For planning purposes:

- One experienced engineer: approximately 4-6 months
- Two experienced engineers: approximately 3-4 months
- Validation plus narrow Phase 1 MVP: approximately 4-7 weeks after validation

Claude CLI and specification-driven development reduce implementation effort, but do not remove source validation, licensing review, data-quality testing, security review, or human acceptance.

## Operating costs

Estimated monthly platform costs from the technical blueprint:

| Stage | Estimated monthly cost |
|---|---:|
| Local proof of concept | $0-$50 |
| Internal MVP | $150-$500 |
| Higher-volume production | $500-$2,000 |

Primary cost drivers:

- VPS and workers
- Managed PostgreSQL and Redis
- Search requests
- LLM extraction
- Enrichment APIs
- CRM plan
- Monitoring and backups
- Human verification time

Government feeds may be inexpensive or free, but human review and source maintenance are recurring operational costs.

## Security and compliance

Required controls include:

- Role-based access control
- Secret manager or protected deployment secrets
- Encryption in transit
- Sensitive-field protection
- SSRF protection
- Source-specific retention policies
- API and AI budget caps
- Audit logs
- Suppression lists
- Deletion workflow
- Backup and restore testing
- Dependency scanning
- Restricted production access
- Adapter kill switch
- Human approval before CRM export

Every source must receive commercial-use and storage review before activation. This repository does not grant permission to use a dataset merely because an adapter can technically access it.

## Observability

Monitor:

- Ingestion job success
- Source freshness
- Source schema failures
- API error and rate-limit events
- Queue length and processing time
- Records per source
- Normalization failures
- Duplicate rate
- False-positive rate
- AI schema failures
- AI and API cost
- Review backlog
- CRM sync failures
- Website audit failures

Nothing should fail silently.

## Deployment

Recommended environments:

- `development` - local services and recorded fixtures
- `staging` - production-like configuration with sandbox or test integrations
- `production` - client-owned accounts, protected secrets, backups, and monitoring

Recommended production components:

- Next.js web deployment
- Dockerized FastAPI application
- Separate background worker container
- Managed PostgreSQL
- Managed Redis or a secured Redis instance
- S3-compatible object storage
- Reverse proxy and TLS
- Centralized logs and monitoring
- Automated database backups

Deployment should be automated through GitHub Actions and require successful quality and migration checks.

## Handover requirements

The client should own:

- GitHub organization and repository
- Hosting and cloud accounts
- Database and backup accounts
- Source and enrichment API accounts
- CRM account
- Domain and DNS
- Monitoring accounts
- Legal and data-licensing records

Handover deliverables:

- Source repository
- Architecture documentation
- Database schema and migration history
- Adapter inventory and source restrictions
- Environment variable template
- Local setup instructions
- Deployment and rollback guide
- Backup and recovery guide
- Monitoring and alert guide
- Data-retention matrix
- Suppression and deletion procedure
- Test suite
- Known limitations
- Operational runbook
- Administrator and reviewer guide

## Decision record

The recommended decision is:

> Validate first. If the Texas experiment succeeds, build the narrow registration-and-licensing radar. Add announcements and website audits only after data quality is proven. Do not build a nationwide scraper.

## Contributing

All changes should:

1. Reference an approved issue or specification.
2. Include tests appropriate to the affected layer.
3. Preserve provenance and retention behaviour.
4. Avoid adding a source without documented terms and ownership.
5. Pass automated quality checks.
6. Receive review for schema, security, and external-source changes.

Use small pull requests and conventional commit messages where practical.

## Licence and confidentiality

This is an internal project for Flex The Brand Studio. Repository access, source-data licences, and redistribution rights must be defined by the client before production use.

Third-party source content remains subject to each provider's terms, licensing, caching, and retention restrictions.

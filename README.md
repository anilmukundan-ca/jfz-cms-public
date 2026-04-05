9# JFZ Contract Management System (JFZ-CMS)

**Jaban's Free Zone FZE — Djibouti, Republic of Djibouti**

A production-grade, cloud-native contract lifecycle management system built for a free zone operator in Djibouti. Manages the full lifecycle of commercial property leases — from draft through approval, final execution, and repository management — across a portfolio of warehouses, offices, cold stores, plots, and business centre units.

---

## Live Application

| Item | Value |
|---|---|
| URL | https://cms.jfzintra.app |
| Access | Google Identity-Aware Proxy (IAP) — authorised users only |
| Cloud Platform | Google Cloud Platform (GCP) |
| Region | europe-west1 (Belgium) |

---

## What This System Does

### Contract Lifecycle Management
- Create draft contracts from Word templates with automatic placeholder substitution
- Submit contracts for approval through a formal approval workflow
- Generate final numbered contracts with an audit trail appended
- Upload signed contract documents (PDF or photo from mobile)
- Track contract status: Draft → Pending Approval → Approved → Active → Expired/Terminated

### Portfolio & Occupancy Management
- 150+ properties across 5 property types (Warehouse, Office, Cold Store, Plot, Business Centre)
- Real-time occupancy dashboard with KPIs: leased area, available area, annual rent revenue, occupancy rate
- Per-property-type breakdown with occupancy percentage bars
- Contracts expiring within 60/30 days highlighted with amber alerts
- Drill-down modals for property history and client contract history

### Interactive Facility Map
- Full satellite/plan image of the JFZ facility displayed as an interactive map
- Colour-coded property pins: green (leased), amber (expiring soon), red (expired), grey (available)
- Hover tooltip on each pin showing property name, client name, status, and days remaining
- Click any pin to go directly to the contract detail page
- Admin-only pin placement mode — drag and drop pins onto the map image
- Real-time pin saving — coordinates stored in PostgreSQL

### Document Repository
- Cloud-hosted document storage (Google Cloud Storage)
- Auto-cleanup of draft/final Word documents when signed contract is uploaded
- Admin housekeeping page to identify and remove orphaned documents
- DPFZA licence document management per client

### Client & Property Management
- Full client database with DPFZA licence tracking
- Property database with type, area, location, and occupancy status
- Import via CSV for bulk data loading
- Contract register with Excel export

### Mobile-First Design
- Responsive hamburger menu on screens under 900px
- All upload inputs support camera, gallery, and file picker equally on Android and iOS
- Progressive Web App (PWA) ready — installable from Chrome to home screen

### Security
- Google Identity-Aware Proxy (IAP) — all access requires an authorised Google account
- Role-based access control: Admin, Approver, Standard, View
- Write-lock system prevents concurrent edits
- All secrets managed via GCP Secret Manager — zero secrets in code

---

## Technology Stack

| Layer | Technology |
|---|---|
| Application | Python 3.11 / Flask 3.0 |
| Database | PostgreSQL 15 (Google Cloud SQL) |
| File Storage | Google Cloud Storage |
| Container | Docker / Google Artifact Registry |
| Hosting | Google Cloud Run (serverless) |
| Authentication | Google Identity-Aware Proxy (IAP) + OAuth 2.0 |
| Secrets | Google Secret Manager |
| DNS | Cloudflare (jfzintra.app) |
| WSGI Server | Gunicorn |
| Document Generation | python-docx |
| Excel Export | openpyxl |
| CI/CD | Google Cloud Build → Cloud Run |
| Version Control | GitHub (this repository) |

---

## Architecture

```
User (Browser/Mobile)
        │
        ▼
  Cloudflare DNS
  cms.jfzintra.app
        │
        ▼
  GCP Load Balancer
  (Static IP: 130.211.38.43)
        │
        ▼
  Identity-Aware Proxy (IAP)
  Google OAuth 2.0 authentication
        │
        ▼
  Cloud Run (jfzcms)
  Flask application — containerised
        │
        ├──► Cloud SQL (PostgreSQL 15)
        │    Contracts, clients, properties, users, map coordinates
        │
        ├──► Cloud Storage (jfz-cms-repository)
        │    Signed contracts, Word documents, templates, licences, facility map
        │
        └──► Secret Manager
             DATABASE_URL, SECRET_KEY, GCS_BUCKET
```

---

## Application Structure

```
jfz-cms/
├── app/
│   ├── __init__.py          # Flask app factory
│   ├── gcs.py               # Google Cloud Storage helpers
│   ├── models/
│   │   ├── db.py            # PostgreSQL connection (psycopg2 RealDictCursor)
│   │   └── schema.sql       # Database schema
│   ├── routes/
│   │   ├── admin.py         # Admin panel — users, settings, housekeeping
│   │   ├── auth.py          # Login/logout
│   │   ├── clients.py       # Client management
│   │   ├── contracts.py     # Contract lifecycle
│   │   ├── dashboard.py     # Home dashboard with KPIs
│   │   ├── facilitymap.py   # Interactive facility map with pin placement
│   │   ├── locks.py         # Write-lock API
│   │   ├── migration.py     # Data migration utilities
│   │   ├── occupancy.py     # Occupancy report with area/revenue stats
│   │   ├── properties.py    # Property management
│   │   ├── repository.py    # Document repository + contract register
│   │   └── help.py          # Help and user manual
│   ├── services/
│   │   ├── auth_helpers.py      # Login decorators
│   │   ├── contract_engine.py   # Word document generation
│   │   └── number_generator.py  # Atomic contract serial number assignment
│   ├── static/
│   │   ├── cms.css          # Application stylesheet
│   │   ├── cms.js           # Client-side scripts
│   │   └── logo.png         # JFZ logo
│   └── templates/           # Jinja2 HTML templates
│       ├── base.html        # Base layout with responsive nav
│       ├── admin/
│       ├── auth/
│       ├── clients/
│       ├── contracts/
│       ├── dashboard/
│       ├── facilitymap/     # Interactive facility map
│       ├── occupancy/
│       ├── properties/
│       ├── register/
│       └── repository/
├── config.py                # Configuration (all secrets from env vars)
├── run.py                   # Application entry point
├── requirements.txt         # Python dependencies
└── Dockerfile               # Container definition
```

---

## User Roles

| Role | Access |
|---|---|
| Admin | Full access — users, settings, housekeeping, pin placement, all contract operations |
| Approver | Can approve/reject contracts submitted for approval |
| Standard | Create and manage contracts, upload documents |
| View | Read-only access to all data |

---

## Contract Number Format

```
JFZ-YYYY-SSSS-TTT-UU-RNN

JFZ     = Jaban's Free Zone
YYYY    = Contract year (from lease start date)
SSSS    = 4-digit serial number (auto-incremented, atomic)
TTT     = Property type code (DWH/OYL/SOL/BC/CLH)
UU      = Unit number
RNN     = Renewal number (R00 = original, R01 = first renewal)

Example: JFZ-2026-0042-DWH-W12-R00
```

---

## Property Types

| Code | Type |
|---|---|
| DWH | Warehouse |
| OYL | Plot (Open Yard / Land) |
| SOL | Office |
| BC | Business Centre |
| CLH | Cold Store |

---

## Key Design Decisions

**PostgreSQL over SQLite** — migrated from a local SQLite database on a Windows LAN server to Cloud SQL PostgreSQL 15 for concurrency, reliability, and cloud-native operation.

**Cloud Run over VM** — serverless deployment scales to zero when idle, eliminating idle VM costs. Gunicorn with 2 workers handles concurrent users.

**IAP over custom auth** — Google Identity-Aware Proxy handles authentication at the infrastructure level, eliminating the need to manage passwords for the primary access layer. CMS application roles provide fine-grained authorisation within the app.

**GCS for documents** — contract documents stored in Google Cloud Storage with object versioning (30-day recovery window) rather than on-disk, enabling stateless containers.

**Atomic serial numbers** — contract serial numbers assigned using `LOCK TABLE serial_counter IN EXCLUSIVE MODE` to prevent duplicate numbers under concurrent load.

**Interactive facility map** — property pins stored as percentage coordinates (x_pct, y_pct) relative to the map image dimensions, making the map fully responsive across all screen sizes.

---

## Deployment

### Build and deploy a new version

```bash
# Edit source files
cd ~/jfz-cms-source

# Commit to GitHub
git add .
git commit -m "describe the change"
git push origin main

# Build Docker image from source
gcloud builds submit ~/jfz-cms-source \
  --tag europe-west1-docker.pkg.dev/jfz-cms-prod/jfz-cms-images/jfzcms:vX.XX \
  --region=europe-west1

# Deploy to Cloud Run
gcloud run deploy jfzcms \
  --image=europe-west1-docker.pkg.dev/jfz-cms-prod/jfz-cms-images/jfzcms:vX.XX \
  --region=europe-west1
```

### Rollback

```bash
gcloud run deploy jfzcms \
  --image=europe-west1-docker.pkg.dev/jfz-cms-prod/jfz-cms-images/jfzcms:vX.PREVIOUS \
  --region=europe-west1
```

### View live logs

```bash
gcloud run services logs read jfzcms \
  --region=europe-west1 \
  --limit=50
```

---

## GCP Infrastructure Reference

| Resource | Details |
|---|---|
| GCP Project | jfz-cms-prod |
| Cloud Run Service | jfzcms — europe-west1 |
| Cloud SQL Instance | jfzcms-db — PostgreSQL 15 — europe-west1-b |
| GCS Bucket | jfz-cms-repository |
| Artifact Registry | europe-west1-docker.pkg.dev/jfz-cms-prod/jfz-cms-images |
| Static IP | 130.211.38.43 |
| Domain | jfzintra.app (Cloudflare — auto-renews April 2027) |

### GCS Folder Structure

```
jfz-cms-repository/
├── drafts/          # Draft Word documents (auto-deleted on signed upload)
├── finals/          # Final Word documents (auto-deleted on signed upload)
├── signed_pdfs/     # Signed contract documents — NEVER deleted
├── templates/       # Contract Word templates (5 files, one per property type)
├── licences/        # Client DPFZA licence documents
└── static/          # Static assets (facility map image)
```

---

## Database

### Key Tables

| Table | Purpose |
|---|---|
| contracts | Contract lifecycle and status |
| clients | Tenant/client details |
| properties | Property portfolio |
| property_map_coords | Facility map pin coordinates (x_pct, y_pct) |
| contract_documents | Uploaded documents (signed contracts, Word files) |
| contract_templates | Word templates per property type |
| serial_counter | Atomic contract serial number counter |
| users | CMS application users and roles |
| activity_log | Full audit trail of all actions |

### Backups
Cloud SQL automatic backups run daily at 02:00 UTC with 7-day retention. Object versioning on GCS provides 30-day recovery for all stored documents.

---

## Monthly Maintenance

```bash
# List old Docker images
gcloud artifacts docker images list \
  europe-west1-docker.pkg.dev/jfz-cms-prod/jfz-cms-images

# Delete old versions (keep current + 1 previous)
gcloud artifacts docker images delete \
  europe-west1-docker.pkg.dev/jfz-cms-prod/jfz-cms-images/jfzcms:v1.0
```

---

## Migration History

The system was originally a Flask + SQLite application running on a Windows 11 LAN server (D:\CMS_Data\). It was migrated to GCP in April 2026 with the following changes:

- SQLite → PostgreSQL 15 (Cloud SQL)
- Local file storage → Google Cloud Storage
- LAN-only access → Cloud Run + IAP (accessible globally with authorisation)
- Manual start → Cloud Run auto-start
- Local backups → Cloud SQL automated backups + GCS object versioning

The Windows system remains stopped as a fallback (D:\CMS_Data\Start_CMS.bat) for a minimum of 90 days post go-live.

---

## Version History

| Version | Key Changes |
|---|---|
| v1.0 – v2.1 | Initial GCP build, GCS integration, PostgreSQL fixes |
| v2.2 | PostgreSQL RealDictCursor compatibility fixes |
| v2.3 | Mobile hamburger navigation menu |
| v2.4 | GCS housekeeping — auto-cleanup on signed upload |
| v2.5 – v2.8 | Admin panel fixes |
| v2.9 | Mobile hamburger menu correctly deployed |
| v2.10 | BEGIN EXCLUSIVE → PostgreSQL LOCK TABLE fix |
| v2.11 | Properties route r[0] → named column fix |
| v2.12 | COUNT(*) fetchone fixes across clients and properties |
| v2.13 – v2.14 | Occupancy page — area, revenue, per-type summary |
| v2.15 | Signed PDF → Signed Contract rename; camera upload |
| v2.16 | Mobile upload — equal camera/gallery/files options |
| v2.17 | All patches consolidated; GitHub repository created |
| v2.18 | First clean build directly from GitHub source |
| v2.19 – v2.23 | Interactive facility map with property pins |

---

## Repositories

| Repository | URL | Visibility |
|---|---|---|
| Source code | https://github.com/anilmukundan-ca/jfz-cms | Private |
| Public showcase | https://github.com/anilmukundan-ca/jfz-cms-public | Public |

---

*Jaban's Free Zone FZE — Djibouti*
*System Administrator: Anil Mukundan (CFO)*

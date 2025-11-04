# ORG.Accountant

A modern, compliance-friendly property accounting system built for Canadian real-estate brokerages and landlords.
Frontend is **Angular**, heavy-lifting services are **Python**, data is stored in **SQL**, and polished printouts are generated as **PDF via LaTeX**.

---

## ✨ Highlights

* **Brokerage-grade accounting**: trust & operating ledgers, bank reconciliation, HST/GST, chart of accounts, vouchers, audit trail.
* **Angular UI**: fast, accessible SPA with role-based UX for admins, bookkeepers, and auditors.
* **Python services**: robust posting engine, reconciliation logic, PDF generation pipeline, scheduled jobs.
* **SQL backend**: transactional integrity, migrations, and reporting views.
* **LaTeX PDFs**: crisp statements, ledgers, and certificates suitable for inspections & clients.
* **API-first**: clean REST (and optional GraphQL) endpoints; typed DTOs and OpenAPI docs.

> Designed with Ontario/RECO workflows in mind (trust vs. operating, monthly reconciliations, supporting documents, durable audit logs).

---

## 👷️ Architecture

```
 ┌────────────────────────┐       HTTPS        ┌────────────────────────┐
 │      Angular        │  <───────────────────> │      API Gateway        │
 │  (SPA + Admin UI)   │                    │  (Angular server-side   │
 │                     │                    │   proxy / reverse proxy)│
 └───────┬───────┘                    └──────┬───────┘
           │                                           │
           │ REST/GraphQL                              │
           ▼                                           ▼
 ┌─────────────────────┐                       ┌─────────────────────┐
 │   Python Service    │  gRPC/HTTP Workers    │  SQL Database        │
 │ (FastAPI + workers) │ <─────────────────> │ (PostgreSQL/MySQL)   │
 │  Posting/Reconcile  │                       │  + migrations        │
 │  PDF (LaTeX)        │                       └─────────────────────┘
 │  Schedulers (RQ/Cel)│
 └───────┬───────┘
           │
           ▼
 ┌──────────────────────────┐
 │  LaTeX toolchain    │
 │  Templates & assets │
 │  Branded PDFs       │
 └──────────────────────────┘
```

---

## 📦 Tech Stack

* **Frontend**: Angular 18+, Angular Material, RxJS, NgRx (optional)
* **API Gateway**: Angular server proxy (dev) or Nginx/Caddy (prod)
* **Backend (Python)**: FastAPI, Pydantic, SQLAlchemy, Alembic, Celery/RQ
* **Database**: PostgreSQL (recommended) or MySQL/MariaDB
* **PDF**: LaTeX (TeX Live), `xelatex`/`lualatex`, `latexmk`
* **Auth**: OAuth2/OIDC (Keycloak/Auth0/Azure AD), JWT
* **Testing**: Jest/Karma (Angular), Pytest (Python)
* **CI/CD**: GitHub Actions (lint, test, build, migrations, PDF smoke)
* **Containers**: Docker & Docker Compose

---

## 🗂️ Repository Layout (suggested)

```
.
├── apps/
│   ├── web/                     # Angular app (SPA)
│   └── api/                     # Python FastAPI service
├── packages/
│   ├── pdf-templates/           # LaTeX classes, templates, assets
│   └── shared/                  # Shared models/types
├── db/
│   ├── migrations/              # Alembic migration scripts
│   └── seeds/                   # Seed data & fixtures
├── ops/
│   ├── docker/                  # Dockerfiles
│   ├── compose/                 # docker-compose.*.yml
│   └── k8s/                     # manifests (optional)
├── docs/                        # Additional documentation
└── Makefile
```

---

## 🚀 Quick Start (Docker)

1. **Prerequisites**

   * Docker Desktop 4.x+
   * Node 20+, PNPM or NPM
   * Python 3.11+
   * TeX Live (or use the LaTeX Docker image)

2. **Environment**
   Create `.env` at repo root (example for PostgreSQL):

   ```
   # Database
   DB_DRIVER=postgresql+psycopg
   DB_HOST=db
   DB_PORT=5432
   DB_NAME=property_accounting
   DB_USER=accounting
   DB_PASSWORD=supersecret

   # API
   API_HOST=0.0.0.0
   API_PORT=8000
   SECRET_KEY=generate_a_long_random_string

   # Auth
   OIDC_ISSUER_URL=https://your-idp.example.com
   OIDC_CLIENT_ID=property-accounting
   OIDC_AUDIENCE=property-accounting-api

   # PDF
   TEX_ENGINE=xelatex
   PDF_OUTPUT_DIR=/data/pdfs
   ```

3. **Boot the stack**

   ```bash
   docker compose -f ops/compose/docker-compose.dev.yml up --build
   ```

   * Angular dev server: `http://localhost:4200`
   * API (FastAPI docs): `http://localhost:8000/docs`
   * DB: `localhost:5432`

4. **Run DB migrations**

   ```bash
   docker compose exec api alembic upgrade head
   ```

5. **Seed sample data (optional)**

   ```bash
   docker compose exec api python -m apps.api.scripts.seed
   ```

---

## 🥮️ Core Concepts

* **Chart of Accounts (COA)**: Configurable codes (e.g., 8520 Advertising, 8760 Auto Insurance) with trust vs. operating flags.
* **Vouchers**: Receipt (RV), Payment (PV), Journal (JV), Commission (CV), Transfers (TR-RV/TR-PV). Auto numbering & posting.
* **Ledgers**: Per-account ledgers + property/entity subledgers; trust ledgers isolate client funds.
* **Bank Reconciliation**: Monthly bank recs (trust & operating), outstanding items, variance checks.
* **Tax**: HST/GST rules, input tax credits, period close.
* **Audit Trail**: Immutable logs of postings, edits, approvals, exports, and PDF hashes.

---

## 🔌 API Overview (FastAPI)

* **Auth**: OAuth2/OIDC Bearer JWT
* **Docs**: `/docs` (Swagger), `/redoc`

Example endpoints:

* `POST /v1/vouchers` — create a voucher
* `POST /v1/postings/{id}/post` — post to GL
* `GET  /v1/ledgers/{account_id}` — ledger lines (paged)
* `GET  /v1/reconciliation/{yyyymm}` — monthly bank rec view
* `POST /v1/pdf/{doc_type}` — generate PDF from LaTeX template (e.g., `ledger`, `bank-recon`, `trust-statement`)

All responses use typed DTOs; errors return structured problem details.

---

## 🖨️ PDF Generation (LaTeX)

* **Engine**: `xelatex` (Unicode & system fonts)
* **Build**: Python workers render Jinja2 → `.tex`, run `latexmk`, store PDF & SHA-256.
* **Branding**: Logo, colours, headers/footers configurable per brokerage.
* **Templates** (examples):

  * `ledger.tex` — General/Trust ledger with booktabs tables
  * `bank_recon.tex` — Summary + outstanding items
  * `statement.tex` — Client statement with period activity

Local test:

```bash
make pdf SAMPLE=ledger
# or
python -m apps.api.pdf.render --template ledger --id 123
```

---

## 📃 Database

* **Recommended**: PostgreSQL 16
* **ORM**: SQLAlchemy 2.x
* **Migrations**: Alembic
* **Keys & Constraints**

  * Strict FKs for vouchers → lines → postings
  * Unique voucher numbers per series
  * Partitioned tables for ledger lines (optional, for scale)
* **Views**

  * `vw_trial_balance`
  * `vw_tax_summary`
  * `vw_trust_position`

Run migrations:

```bash
alembic revision -m "feat: add trust ledger indices"
alembic upgrade head
```

---

## 🔐 Security & Compliance

* **AuthZ**: Role-based access (Admin, Bookkeeper, Auditor, Read-only)
* **PII**: Encrypted at rest (DB TDE or app-level field encryption)
* **Secrets**: `.env` + secret manager (AWS/GCP/Azure)
* **Audit**: Append-only event store with actor, timestamp, checksum
* **Backups**: PITR capable (e.g., pgBackRest); verify restores monthly
* **RECO/TRESA alignment (Ontario)**:

  * Trust vs. Operating segregation
  * Monthly reconciliations with zero-balance months supported
  * Exportable supporting documents & ledgers as PDF

---

## 🤪 Testing

**Angular**

```bash
cd apps/web
npm ci
npm run test
npm run lint
```

**Python**

```bash
cd apps/api
pip install -e .[dev]
pytest -q
ruff check .
mypy apps/api
```

**PDF smoke**

```bash
pytest tests/pdf --maxfail=1
```

---

## 🛠️ Development Tips

* Use Angular proxy (`proxy.conf.json`) to forward `/api` → `http://localhost:8000`.
* Enable Hot Module Replacement for rapid UI iteration.
* Keep LaTeX templates small and compose with `\input{parts/...}`.
* Prefer idempotent API mutations; retry-safe posting.
* Use DB transactions for multi-step postings (commit or roll back atomically).

---

## 📤 Exports & Integrations

* **CSV/Excel**: Trial balance, GL dump, tax summary
* **Bank Feeds**: OFX/CSV importers, rule-based matching (optional)
* **Storage**: Local filesystem or S3-compatible bucket for PDFs & attachments
* **Webhooks**: Integration points for external audit or data sync

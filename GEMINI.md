# GEMINI.md - TightWallet

## Project Overview
**TightWallet** is a personal finance management application built on a foundation of **real double-entry bookkeeping** (inspired by GnuCash). It bridges the gap between accounting rigor and modern user experience by offering two distinct interfaces:
- **Classic Mode**: A simplified, everyday UX for non-technical users.
- **Advanced Mode**: A full accounting interface with account trees, ledgers, and complex splits.

The project is designed as **web-first** (Angular) and **mobile-first** (Flutter) with a strong emphasis on **offline-first** capabilities and seamless synchronization.

## Architecture & Tech Stack (Recommended)
Based on the current technical documentation (`doc/TightWallet_Tecnologie_Consigliate.md`):

- **Backend**: .NET 8/9 (ASP.NET Core)
  - **ORM**: Entity Framework Core
  - **Database**: PostgreSQL (with JSONB support for flexibility)
  - **Job Runner**: Hangfire (for recurring transactions and import pipelines)
- **Web App**: Angular 17+ (Material or PrimeNG)
- **Mobile App**: Flutter (Offline-first with SQLite/Drift)
- **Communication**: REST API (OpenAPI/Swagger)

## Core Principles
1. **Double-Entry Consistency**: Every transaction must balance (Debits = Credits).
2. **Unified Data Model**: Classic and Advanced modes share the same database; only the view changes.
3. **Offline-First**: The mobile app must remain functional without internet access, utilizing an idempotent sync protocol.
4. **Supervised Automation**: Recurring transactions are generated as "Expected Instances" in an Inbox for user approval/modification.
5. **Multi-Currency**: Accounts have specific currencies; a base currency is used for aggregate reporting.

## Key Features
- **Fast Entry**: Optimized workflows for expenses, income, and transfers.
- **Import Pipeline**: Robust support for migrating from GnuCash, Money Manager, and CSV/Excel.
- **Composable Dashboard**: Customizable widgets for KPIs (Net Savings, Burn Rate, etc.).
- **Payee Normalization**: Canonical names and aliases for clean reporting.
- **Utilities Tracking**: Optional tracking of consumption metrics (m³, kWh) alongside costs.

## Database Schema Highlights
The project uses a detailed PostgreSQL schema featuring:
- `ledger_accounts`: Hierarchical chart of accounts (Assets, Liabilities, Equity, Income, Expenses).
- `transactions` & `splits`: The core double-entry engine.
- `scheduled_rules` & `expected_instances`: For recurring payments.
- `change_events` & `sync_cursors`: For offline synchronization.

## Development Roadmap
- **Phase 1 (Core)**: Account tree, double-entry engine, importers, and basic UI modes.
- **Phase 2 (Differentiation)**: Scheduled rules/Inbox, budget goals, and offline sync.
- **Phase 3 (Expansion)**: Advanced reporting, AI-assisted chat/analysis, and open banking.

## Key Documentation Files
- `README.md`: High-level overview and proposed repository structure.
- `doc/TightWallet_Documento_Esaustivo_v5.md`: The most comprehensive product specification.
- `doc/TightWallet_Tecnologie_Consigliate.md`: Technical stack recommendations and architecture notes.
- `doc/TightWallet_Documento_v4_Classic_vs_Advanced.md`: Detailed comparison of UI modes.

## Asset Management Standards
- **Image Extraction**: When extracting graphical elements from templates or mockups:
  - **Precision**: Use contour detection or exact bounding boxes to ensure zero white/background borders.
  - **Transparency**: Always convert extracted PNGs to RGBA and replace background colors (e.g., template canvas colors) with full transparency, especially for rounded corners, circular icons, or non-rectangular mockups.
  - **Quality**: Prefer high-resolution source files (e.g., `logo.jpeg`) over upscaling small elements from a template when generating standard app icon sizes.
  - **Consistency**: Maintain original aspect ratios unless a specific target dimension (like a 1024x500 Feature Graphic) is required.

## Usage
This repository currently serves as the **design and specification hub** for TightWallet. Use the documents in `doc/` to guide the implementation of the backend, web, and mobile components as they are scaffolded.

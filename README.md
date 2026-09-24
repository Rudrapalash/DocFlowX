# DocFlowX

DocFlowX is a web portal where users sign in with Microsoft Entra ID, register ingestion sources, and sync content into a target SharePoint folder.

## Objective

Build a user-centric ingestion and sync platform with the following rule:

- All data fetched from any source must be converted to `.docx` first.
- Only after conversion, the `.docx` output is uploaded to the target SharePoint folder.

## Tech Stack

- Frontend: React
- Backend API: Python FastAPI
- Database: PostgreSQL
- Authentication: Microsoft Entra ID (OAuth2 / OpenID Connect, delegated user context)
- Integrations:
  - Microsoft Graph API for SharePoint read/write
  - Azure DevOps REST APIs for repository/wiki content fetch

## High-Level Architecture

- React web portal for login, source management, scheduling configuration, and manual sync actions.
- FastAPI service for token-aware ingestion, content normalization, DOCX conversion orchestration, scheduling execution, and SharePoint upload.
- PostgreSQL for persistent storage of users, source registry, schedules, sync job history, and status metadata.

## Supported Sources

- Azure DevOps repository links.
- Azure DevOps wiki links.
- SharePoint file/folder links.

## Identity and Access Model

- User login is through Microsoft Entra ID.
- Source access is delegated and tied to the logged-in user identity.
- Azure DevOps data is read using the user delegated access token.
- SharePoint data is read and written using Microsoft Graph delegated access for the same user.
- Each user can only view and manage their own enlisted sources.

## Source Registry Per User

For every logged-in user, maintain a source list with:

- `sourceId`
- `ownerUserId` (Entra user/object ID)
- `sourceType` (`azureRepo`, `azureWiki`, `sharePoint`)
- `sourceUrl`
- `scheduleEnabled`
- `scheduleInterval`
- `lastSyncStatus`
- `lastSyncTime`
- `lastError`

## Sync Modes

- Manual sync:
  - User triggers sync on demand for one source or all sources.
- Scheduled auto-sync:
  - User sets interval-based schedules (for example 15 minutes, hourly, 6-hourly, daily).
  - Background scheduler triggers sync jobs per source.

## Required Processing Pipeline

Every sync run must follow this fixed pipeline:

1. Fetch data from source using delegated user token.
2. Normalize extracted content.
3. Convert normalized output into `.docx`.
4. Upload generated `.docx` file(s) to configured target SharePoint folder.
5. Store sync job result (success/failure, timestamp, error/logs).

## DOCX Conversion Rule (Mandatory)

- No raw ZIP/JSON/Markdown/TXT should be uploaded directly to target SharePoint.
- Each source output must be represented as one or more `.docx` documents.
- File naming should be deterministic and traceable (include source label/type and timestamp).

## Target SharePoint Upload

- Upload destination is configured by the logged-in user (site/drive/folder).
- Uploaded `.docx` files must be accessible with the same Entra identity context used for login.

## Non-Functional Expectations

- Multi-user isolation for data and source registry.
- Token expiry handling and re-auth flow.
- Audit logs for sync history.
- Retry handling for transient API failures.
- Security controls for token storage and API protection.

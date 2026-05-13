# OSS Attachment Control PCF

Two Power Apps Component Framework (PCF) controls for document attachment management built against the `os_documentconfiguration` and `os_documentmetadata` Dataverse entities.

---

## Controls

### 1. `AttachmentUploadControl` (OSS.AttachmentUploadControl)

A **field PCF control** added to any entity form. It:

- Reads `os_documentconfiguration` records filtered by the current entity's logical name and `os_showdocument = true`.
- Renders a card for each configured document type, indicating whether it is **Required** or **Optional**.
- Allows users to **upload** files (base64-encoded) directly into `os_documentmetadata`.
- Shows already-uploaded files per document type with **Download** and **Delete** actions.
- Validates file extensions against `os_extensionavailable` before uploading.
- Warns users to save the record first if the record ID is not yet available.

**Data written to `os_documentmetadata`:**

| Field | Value |
|---|---|
| `os_filename` | Original file name |
| `os_extension` | File extension (e.g. `.pdf`) |
| `os_contenttype` | MIME type |
| `os_base64` | Base64-encoded file content |
| `os_entityname` | Entity logical name |
| `os_requestname` | Record ID (GUID) |
| `os_documentname` | Document type name from configuration |

---

### 2. `DocumentSubgridControl` (OSS.DocumentSubgridControl)

A **field PCF control** that renders as a full-featured subgrid. It:

- Retrieves all `os_documentmetadata` records for the current entity/record.
- Displays them in a sortable, searchable, paginated table.
- Columns: **Document Type**, **File Name**, **Extension**, **Content Type**, **Upload Date**, **Actions**.
- **Download** – streams base64 content as a file download (or opens `os_url` if present).
- **Delete** single row or **bulk delete** selected rows.
- **Search** across all text columns.
- **Column sort** (click column header to cycle asc → desc → none).
- **Pagination** with configurable page size (default: 10 rows/page).

---

## Project Structure

```
AttachmentControlPCF/
├── AttachmentUploadControl/          # Upload PCF project
│   ├── package.json
│   ├── tsconfig.json
│   └── AttachmentUploadControl/
│       ├── ControlManifest.Input.xml
│       ├── index.ts
│       ├── generated/
│       │   └── ManifestTypes.d.ts
│       └── css/
│           └── AttachmentUploadControl.css
├── DocumentSubgridControl/           # Subgrid PCF project
│   ├── package.json
│   ├── tsconfig.json
│   └── DocumentSubgridControl/
│       ├── ControlManifest.Input.xml
│       ├── index.ts
│       ├── generated/
│       │   └── ManifestTypes.d.ts
│       └── css/
│           └── DocumentSubgridControl.css
└── Solutions/                        # Power Platform solution wrapper
    ├── solution.cdsproj
    └── src/
        └── Other/
            └── Solution.xml
```

---

## Prerequisites

- Node.js ≥ 16
- [Power Platform CLI (`pac`)](https://aka.ms/PowerPlatformCLI)
- npm or yarn

---

## Build & Test Locally

```bash
# AttachmentUploadControl
cd AttachmentUploadControl
npm install
npm run build        # production build
npm start            # local test harness (http://localhost:8181)

# DocumentSubgridControl
cd ../DocumentSubgridControl
npm install
npm run build
npm start
```

---

## Deploy to Power Platform

### Option A – pac CLI (recommended)

```bash
# Build both controls
cd AttachmentUploadControl && npm run build
cd ../DocumentSubgridControl && npm run build

# Authenticate
pac auth create --url https://<your-org>.crm.dynamics.com

# Push each control
pac pcf push --publisher-prefix os --environment <env-url>  # run inside each control folder
```

### Option B – Solution import

1. Build both controls with `npm run build`.
2. Package the solution using Power Platform Build Tools or `pac solution pack`.
3. Import the `.zip` into your environment via the Maker Portal or `pac solution import`.

---

## Adding Controls to a Form

### AttachmentUploadControl

1. Open the target entity form in the Maker Portal.
2. Add (or use an existing) **Single Line of Text** field.
3. Select the field → **Change component** → choose **OSS.AttachmentUploadControl**.
4. Save & Publish.

### DocumentSubgridControl

1. Add a **Single Line of Text** field (or an existing one) to the form.
2. Select it → **Change component** → choose **OSS.DocumentSubgridControl**.
3. Save & Publish.

---

## Entity Schema Reference

### DocumentConfiguration (`os_documentconfiguration`)

| Schema Name | Logical Name | Description |
|---|---|---|
| `os_documentconfigurationid` | Primary Key | |
| `os_documentname` | Document Name | Human-readable label |
| `os_entitylogicalname` | Entity Logical Name | Filters config per entity |
| `os_ismandatory` | Is Mandatory | Required flag |
| `os_showdocument` | Show Document | Visibility flag |
| `os_extensionavailable` | Extension Available | Comma-separated allowed extensions |

### DocumentMetadata (`os_documentmetadata`)

| Schema Name | Logical Name | Description |
|---|---|---|
| `os_documentmetadataid` | Primary Key | |
| `os_filename` | File Name | |
| `os_extension` | Extension | e.g. `.pdf` |
| `os_contenttype` | Content Type | MIME type |
| `os_base64` | Base64 Content | File bytes as base64 |
| `os_entityname` | Entity Name | Parent entity logical name |
| `os_requestname` | Request Name | Parent record ID (GUID) |
| `os_documentname` | Document Name | Links back to config |
| `os_url` | URL | Optional direct file URL |

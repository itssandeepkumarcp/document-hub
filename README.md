# New project: Document Hub.


### Current problem 

client has a lot of documents (mostly scanned pdfs or images of documents) which are grouped by projects. Each project can have many documents. Currently they are stored on personal PCs, some on google drive.

### Requirements

Client wants a system to store all documents by project, add categories, notes (description for each document). Document Hub should be accessible by company employees. Each employee can add documents, but only managers(admins) can delete them.

For first stage of the project client wants to keep it simple:

●	Ability to create/edit/delete projects

●	Ability to create/edit/delete categories (managed by admins)

●	Ability to upload project documents and set categories.

●	Search documents by project, categories, notes, and date of document upload.

However, the client would like to investigate a possibility to search by document content. All documents are printed text, there is no hand written text. Scanned documents are text only, no graphs or pictures.


### The task for Solution Architect is to provide a plan for project consisting of:

●	Data Structure which would represent required database tables (in https://app.diagrams.net , https://lucid.app or similar)

●	Suggestion for data storage solution in cloud (Azure)

●	Suggestion for OCR solution which could convert pdfs/images to searchable text.

●	High level infrastructure diagram consisting of servers for backend, database, serverless solutions if any needed.


### Once done, please send us links or files to

●	Data structure diagram

●	High level infrastructure

●	Any other documents you would create

We will ask you to present your solution in english.

---

## Solution Deliverables

### 📋 Core Documentation
- **[architecture_doc.md](architecture_doc.md)** - Complete solution architecture document with 12 comprehensive sections covering database design, Azure storage, OCR solution, infrastructure, cost estimation, security, and implementation timeline

### 📊 Diagrams

#### Database & Infrastructure
- **[db_schema_diagram.mermaid](db_schema_diagram.mermaid)** - Complete database ER diagram with all tables and relationships
- **[infrastructure_diagram.mermaid](infrastructure_diagram.mermaid)** - High-level Azure infrastructure architecture
- **[deployment_diagram.mermaid](deployment_diagram.mermaid)** - Detailed production deployment with service specifications

#### Sequence Diagrams (Process Flows)
- **[sequence_diagram_upload.mermaid](sequence_diagram_upload.mermaid)** - Document upload and asynchronous OCR processing flow
- **[sequence_diagram_search.mermaid](sequence_diagram_search.mermaid)** - Document search and secure download flow
- **[sequence_diagram_admin.mermaid](sequence_diagram_admin.mermaid)** - Admin operations and role-based authorization flow

### 📖 Reference
- **[DIAGRAMS_README.md](DIAGRAMS_README.md)** - Guide to viewing and working with all diagrams
- **[VALIDATION_REPORT.md](VALIDATION_REPORT.md)** - Compliance verification against requirements

---

## Quick Links

| Requirement | Delivered Solution |
|------------|-------------------|
| Data Structure | [db_schema_diagram.mermaid](db_schema_diagram.mermaid) |
| Azure Storage Solution | [architecture_doc.md - Section 2](architecture_doc.md#2-data-storage-solution-azure) |
| OCR Solution | [architecture_doc.md - Section 3](architecture_doc.md#3-ocr-solution) |
| Infrastructure Diagram | [infrastructure_diagram.mermaid](infrastructure_diagram.mermaid) + [deployment_diagram.mermaid](deployment_diagram.mermaid) |
| Process Flows | [Sequence Diagrams](DIAGRAMS_README.md#4-sequence-diagrams) |

---

## How to View Diagrams

All diagrams use **Mermaid** syntax and are automatically rendered on GitHub. You can also:
- View/edit at [Mermaid Live Editor](https://mermaid.live/)
- Use VS Code with Mermaid extension
- See [DIAGRAMS_README.md](DIAGRAMS_README.md) for more options


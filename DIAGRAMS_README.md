# Document Hub - Diagrams Reference

This directory contains comprehensive architectural diagrams for the Document Hub system.

## Available Diagrams

### 1. Database Schema
**File:** `db_schema_diagram.mermaid`  
**Type:** Entity-Relationship Diagram (ERD)  
**Purpose:** Complete database structure showing all tables, relationships, and fields

**Tables Included:**
- Users (employee and manager roles)
- Projects (document grouping)
- Categories (admin-managed classification)
- Documents (file metadata and OCR text)
- DocumentCategories (many-to-many relationship)
- AuditLog (complete audit trail)

---

### 2. Infrastructure Architecture
**File:** `infrastructure_diagram.mermaid`  
**Type:** High-Level Architecture Diagram  
**Purpose:** Overview of Azure cloud services and their connections

**Components:**
- Frontend: Web Application (React/Angular)
- Backend: Azure App Service API
- Serverless: Azure Functions (OCR, Search Indexing)
- Data: Azure SQL, Blob Storage, Cognitive Search
- AI: Azure Document Intelligence (OCR)
- Security: Azure AD B2C, Key Vault, Application Insights

---

### 3. Deployment Architecture
**File:** `deployment_diagram.mermaid`  
**Type:** Detailed Deployment Diagram  
**Purpose:** Production environment deployment with specific Azure service configurations

**Details:**
- Complete service specifications (tiers, instances, storage)
- Network topology (VNet, Private Endpoints)
- Security layers (WAF, DDoS, encryption)
- Monitoring and logging setup
- DevOps pipeline integration
- Specific Azure regions and redundancy

---

### 4. Sequence Diagrams

#### 4.1 Document Upload Flow
**File:** `sequence_diagram_upload.mermaid`  
**Type:** Sequence Diagram  
**Purpose:** Step-by-step document upload and OCR processing flow

**Actors:**
- User, Web Application, Azure AD B2C
- API Backend, Azure SQL Database
- Blob Storage, Service Bus Queue
- OCR Function, Document Intelligence, Cognitive Search

**Key Flows:**
1. User authentication and file upload
2. Validation and storage
3. Asynchronous OCR processing
4. Search indexing

---

#### 4.2 Document Search Flow
**File:** `sequence_diagram_search.mermaid`  
**Type:** Sequence Diagram  
**Purpose:** Document search and download process

**Key Flows:**
1. Search query with filters
2. Full-text search across document content
3. Results ranking and highlighting
4. Secure document download with SAS tokens
5. Permission validation

---

#### 4.3 Admin Operations Flow
**File:** `sequence_diagram_admin.mermaid`  
**Type:** Sequence Diagram  
**Purpose:** Administrative operations (manager/admin only)

**Key Flows:**
1. Category management (create/edit/delete)
2. Document deletion (soft delete with audit trail)
3. Project management
4. Role-based authorization checks

---

## How to View These Diagrams

### Option 1: GitHub (Automatic Rendering)
GitHub automatically renders Mermaid diagrams in `.mermaid` files and markdown code blocks.

### Option 2: Mermaid Live Editor
1. Visit [Mermaid Live Editor](https://mermaid.live/)
2. Copy the content of any `.mermaid` file
3. Paste into the editor to view and edit

### Option 3: VS Code
Install the "Markdown Preview Mermaid Support" extension:
- Extension ID: `bierner.markdown-mermaid`
- Preview diagrams directly in VS Code

### Option 4: IntelliJ/WebStorm
Built-in Mermaid support in JetBrains IDEs (2021.1+)

### Option 5: Documentation Tools
- Docusaurus: Native Mermaid support
- MkDocs: Use `mkdocs-mermaid2-plugin`
- Confluence: Use Mermaid macro

---

## Diagram Syntax

All diagrams use Mermaid syntax (https://mermaid.js.org/), which is:
- ✅ Version-controllable (plain text)
- ✅ Easy to update and maintain
- ✅ Rendered automatically by GitHub, GitLab, Azure DevOps
- ✅ Supported by major documentation platforms
- ✅ Can be exported to PNG/SVG

---

## Integration with Architecture Document

The diagrams complement the detailed architecture documentation in `architecture_doc.md`:

| Diagram | Related Architecture Doc Section |
|---------|----------------------------------|
| db_schema_diagram.mermaid | Section 1: Database Design |
| infrastructure_diagram.mermaid | Section 4: High-Level Infrastructure |
| deployment_diagram.mermaid | Section 4 & 6: Infrastructure + Scalability |
| sequence_diagram_upload.mermaid | Section 5.1: Document Upload Flow |
| sequence_diagram_search.mermaid | Section 5.2: Document Search Flow |
| sequence_diagram_admin.mermaid | Section 4.2 & 4.5: API Layer + Authorization |

---

## Updating Diagrams

When updating diagrams:

1. **Test syntax** at [Mermaid Live Editor](https://mermaid.live/) first
2. **Keep consistent** with architecture_doc.md
3. **Update this README** if adding new diagrams
4. **Version control** - commit with descriptive messages
5. **Validate rendering** - check on GitHub after push

---

## Export Options

To export diagrams to images:

### Using Mermaid CLI
```bash
npm install -g @mermaid-js/mermaid-cli
mmdc -i db_schema_diagram.mermaid -o db_schema.png
```

### Using Mermaid Live Editor
1. Open diagram in live editor
2. Click "Actions" → "PNG" or "SVG"
3. Download high-resolution image

### Using VS Code
1. Install "Markdown PDF" extension
2. Right-click diagram in preview
3. "Export to PDF/PNG"

---

## Diagram Best Practices

✅ **DO:**
- Use clear, descriptive labels
- Keep diagrams focused and readable
- Use subgraphs for logical grouping
- Add notes for complex flows
- Style important components

❌ **DON'T:**
- Overcrowd diagrams with too much detail
- Use inconsistent naming with architecture doc
- Forget to update when architecture changes
- Mix different abstraction levels in one diagram

---

## Questions or Issues?

For questions about the diagrams or to request new diagram types:
- Open an issue in the repository
- Contact the solution architect team
- Refer to the main `architecture_doc.md` for detailed explanations

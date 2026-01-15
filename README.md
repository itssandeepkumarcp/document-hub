# Document Hub - Solution Architecture Document

## Executive Summary

This document outlines the technical architecture for the Document Hub system, a cloud-based document management solution hosted on Microsoft Azure. The system enables secure storage, categorization, and searchable access to project documents with OCR capabilities for scanned content.

---

## 1. Database Design

### 1.1 Relational Database Structure

**Recommended Solution: Azure SQL Database** (Standard Tier S2-S4)

#### Core Tables:

**Users**
- Stores employee information and role-based access
- Roles: `employee` (can upload) and `manager` (can delete)
- Integration with Azure AD B2C for authentication

**Projects**
- Central entity for grouping documents
- Soft delete support (is_active flag)
- Tracks creator and timestamps

**Categories**
- Admin-managed classification system
- Supports multiple categories per document
- Soft delete support

**Documents**
- Core document metadata
- References blob storage location
- Contains OCR extracted text for search
- Soft delete with audit trail (deleted_by, deleted_at)

**DocumentCategories**
- Many-to-many relationship between Documents and Categories
- Allows multiple categories per document

**AuditLog**
- Complete audit trail for compliance
- Tracks all CRUD operations
- Stores JSON snapshots of changes

### 1.2 Alternative Database Option

**Azure PostgreSQL Flexible Server** - If open-source preference or advanced full-text search features needed

---

## 2. Data Storage Solution (Azure)

### 2.1 Azure Blob Storage

**Configuration:**
- **Account Type:** StorageV2 (General Purpose v2)
- **Redundancy:** LRS (Locally Redundant) or ZRS (Zone Redundant) based on compliance needs
- **Access Tiers:**
  - Hot tier: Recently uploaded documents (0-90 days)
  - Cool tier: Older documents (automated lifecycle policy)
  
**Container Structure:**
```
documents/
  ├── {project_id}/
  │   ├── {document_id}/
  │   │   ├── original.pdf
  │   │   └── thumbnail.png (optional)
```

**Security:**
- Private containers (no anonymous access)
- Shared Access Signatures (SAS) with expiration for temporary access
- Customer-managed encryption keys (optional)
- Soft delete enabled (retention: 30 days)

**Benefits:**
- Unlimited scalability
- Cost-effective ($0.018/GB for hot tier)
- Built-in redundancy and durability (99.999999999% durability)
- Integration with Azure CDN for faster document delivery
- Lifecycle management for automatic tier transitions

---

## 3. OCR Solution

### 3.1 Recommended: Azure AI Document Intelligence (Form Recognizer)

**Why This Solution:**
- Purpose-built for document OCR
- Handles multiple formats: PDF, JPEG, PNG, TIFF, BMP
- High accuracy for printed text (98%+)
- Preserves document structure and layout
- Supports batch processing
- Pay-per-page pricing model

**Pricing:** ~$1.50 per 1,000 pages (Read API)

**API Endpoint:** Read API (Asynchronous)

**Implementation Flow:**
1. Document uploaded to Blob Storage
2. Azure Function triggered via Storage Queue
3. Function calls Document Intelligence Read API
4. Async processing (polling or callback)
5. Extract text and confidence scores
6. Store OCR text in SQL database
7. Index text in Azure Cognitive Search

**Sample Output:**
```json
{
  "content": "Full extracted text...",
  "pages": [
    {
      "pageNumber": 1,
      "lines": [
        {
          "content": "Document title here",
          "boundingBox": [...],
          "words": [...]
        }
      ]
    }
  ]
}
```

### 3.2 Alternative Solutions

**Option B: Azure Cognitive Services Computer Vision (Read API)**
- Similar to Document Intelligence but simpler
- Good for straightforward text extraction
- Lower cost for basic scenarios

**Option C: Azure OpenAI with GPT-4 Vision (Future Enhancement)**
- Can understand context and structure
- More expensive but adds intelligence
- Good for complex document understanding

---

## 4. High-Level Infrastructure Architecture

### 4.1 Frontend Layer

**Technology:** React or Angular SPA
**Hosting:** Azure Static Web Apps or Azure App Service
**Features:**
- Responsive design for desktop/mobile
- File upload with drag-drop
- Search interface with filters
- Document preview

### 4.2 API Layer

**Azure App Service (Web App)**
- **Technology:** ASP.NET Core 8 or Node.js (Express)
- **Tier:** Standard S1-S2 (can scale up/out)
- **Features:**
  - RESTful API endpoints
  - JWT-based authentication
  - Role-based authorization
  - Input validation and sanitization
  - Rate limiting

**Key API Endpoints:**
```
POST   /api/projects
GET    /api/projects
PUT    /api/projects/{id}
DELETE /api/projects/{id}

POST   /api/categories (Admin only)
GET    /api/categories
PUT    /api/categories/{id} (Admin only)
DELETE /api/categories/{id} (Admin only)

POST   /api/documents/upload
GET    /api/documents/search
GET    /api/documents/{id}
PUT    /api/documents/{id}
DELETE /api/documents/{id} (Manager only)
GET    /api/documents/{id}/download
```

### 4.3 Serverless Layer

**Azure Functions (Consumption or Premium Plan)**

**Function 1: OCR Processing Function**
- **Trigger:** Azure Storage Queue message
- **Process:**
  1. Retrieve document from Blob Storage
  2. Call Azure Document Intelligence API
  3. Poll for completion (or use callback)
  4. Store extracted text in SQL database
  5. Update search index
  6. Mark document as OCR-processed
- **Timeout:** 10 minutes (Premium plan if needed)

**Function 2: Search Indexing Function**
- **Trigger:** Timer (every 5-15 minutes) or Service Bus
- **Process:**
  1. Query documents pending indexing
  2. Push to Azure Cognitive Search
  3. Update indexed status

**Function 3: Cleanup Function (Optional)**
- **Trigger:** Timer (daily)
- **Process:**
  1. Delete soft-deleted documents after retention period
  2. Move old documents to cool storage tier

### 4.4 Search Layer

**Azure Cognitive Search**
- **Tier:** Basic or Standard S1
- **Features:**
  - Full-text search with relevance scoring
  - Faceted search (by project, category, date)
  - Autocomplete and suggestions
  - Custom analyzers for document text
  
**Index Schema:**
```json
{
  "name": "documents-index",
  "fields": [
    {"name": "document_id", "type": "Edm.Int32", "key": true},
    {"name": "project_name", "type": "Edm.String", "filterable": true},
    {"name": "document_name", "type": "Edm.String", "searchable": true},
    {"name": "categories", "type": "Collection(Edm.String)", "filterable": true},
    {"name": "notes", "type": "Edm.String", "searchable": true},
    {"name": "ocr_text", "type": "Edm.String", "searchable": true},
    {"name": "uploaded_at", "type": "Edm.DateTimeOffset", "sortable": true, "filterable": true}
  ]
}
```

### 4.5 Authentication & Authorization

**Azure AD B2C**
- Single Sign-On (SSO)
- Multi-factor authentication (MFA)
- User self-service (password reset)
- Integration with existing company AD

**Authorization Model:**
- Role-based access control (RBAC)
- Two roles: Employee, Manager
- Implemented via JWT claims

### 4.6 Monitoring & Security

**Application Insights**
- Real-time performance monitoring
- Exception tracking
- Custom telemetry and logging
- Usage analytics

**Azure Key Vault**
- Storage account keys
- Database connection strings
- OCR API keys
- JWT signing keys

**Azure Front Door / Application Gateway**
- Web Application Firewall (WAF)
- DDoS protection
- SSL/TLS termination
- Geographic load balancing

---

## 5. Data Flow Diagrams

### 5.1 Document Upload Flow

```
1. User uploads document via Web UI
2. Frontend sends file to API with metadata
3. API validates user permissions and file
4. API generates unique document_id
5. API uploads file to Blob Storage
6. API creates document record in SQL (ocr_processed=false)
7. API sends message to Storage Queue/Service Bus
8. API returns success to user
9. [Async] OCR Function triggered by queue message
10. [Async] Function processes document with Document Intelligence
11. [Async] Function updates SQL with OCR text
12. [Async] Function updates search index
13. [Async] Document marked as OCR-processed
```

### 5.2 Document Search Flow

```
1. User enters search query with filters
2. Frontend sends request to API
3. API validates user authentication
4. API queries Azure Cognitive Search
   - Full-text search on: document_name, notes, ocr_text
   - Filters on: project_id, categories, date range
5. Search returns ranked results with highlights
6. API enriches results with database metadata
7. API returns results to frontend
8. User can download/preview documents
```

---

## 6. Scalability & Performance Considerations

### 6.1 Expected Load Handling

- **Concurrent Users:** 50-100 (can scale to 500+)
- **Documents:** 10,000-100,000+ documents
- **Storage:** 1TB-10TB+ capacity
- **OCR Processing:** 1,000-5,000 pages/day

### 6.2 Scaling Strategy

**Horizontal Scaling:**
- App Service: Auto-scale based on CPU/memory (2-10 instances)
- Azure Functions: Automatic scaling (consumption plan)
- Azure SQL: Read replicas for heavy search workloads

**Vertical Scaling:**
- Database: Upgrade from S2 → S4 → P1 as needed
- Cognitive Search: Basic → Standard S1 → S2

**Caching:**
- Azure Redis Cache for frequent queries
- CDN caching for document thumbnails
- API response caching (categories, projects list)

---

## 7. Cost Estimation (Monthly - Medium Scale)

| Service | Specification | Est. Cost |
|---------|--------------|-----------|
| Azure App Service | Standard S2 | $146 |
| Azure SQL Database | Standard S2 (50 DTU) | $150 |
| Azure Blob Storage | 1TB (Hot tier) | $18 |
| Azure Functions | Consumption (1M executions) | $0.20 |
| Azure Document Intelligence | 10,000 pages/month | $15 |
| Azure Cognitive Search | Basic tier | $75 |
| Azure AD B2C | 50 users | Free |
| Application Insights | 5GB/month | $12 |
| Azure Key Vault | Secrets storage | $0.15 |
| **Total Estimated** | | **~$416/month** |

*Note: Costs scale with usage. Production may require higher tiers.*

---

## 8. Security Best Practices

1. **Data Encryption:**
   - In-transit: TLS 1.2+ for all communications
   - At-rest: Azure Storage encryption (256-bit AES)

2. **Access Control:**
   - Least privilege principle
   - Managed identities for Azure services
   - No hardcoded credentials

3. **Network Security:**
   - Virtual Network (VNet) integration
   - Private endpoints for SQL and Storage
   - NSG rules for traffic filtering

4. **Compliance:**
   - GDPR compliance (data residency, right to deletion)
   - Audit logging for all operations
   - Data retention policies

5. **Backup & Disaster Recovery:**
   - SQL automated backups (7-35 days retention)
   - Blob soft delete (30 days)
   - Geo-redundant storage option
   - RPO: < 1 hour, RTO: < 4 hours

---

## 9. Implementation Phases

### Phase 1: MVP (4-6 weeks)
- Basic CRUD for projects and categories
- Document upload to Blob Storage
- User authentication with Azure AD B2C
- Simple metadata search (no OCR)

### Phase 2: OCR Integration (3-4 weeks)
- Azure Function for OCR processing
- Document Intelligence API integration
- Store extracted text in database

### Phase 3: Advanced Search (2-3 weeks)
- Azure Cognitive Search integration
- Full-text search on document content
- Search filters and facets

### Phase 4: Polish & Optimization (2-3 weeks)
- Performance optimization
- Monitoring and alerts
- User feedback and refinements

**Total Timeline: 11-16 weeks**

---

## 10. Risk & Mitigation

| Risk | Impact | Mitigation |
|------|--------|-----------|
| OCR accuracy issues | Medium | Use Document Intelligence high-quality mode; manual review option |
| Large file processing timeouts | Medium | Use durable functions; chunk large files |
| Storage costs exceed budget | Low | Implement lifecycle policies; compress files |
| Search performance degradation | Medium | Implement caching; optimize indexes; scale search tier |
| Concurrent upload issues | Low | Implement queue-based processing; retry logic |

---

## 11. Future Enhancements

1. **AI-Powered Features:**
   - Automatic document categorization using ML
   - Smart tagging based on content
   - Document similarity detection

2. **Collaboration:**
   - Document version control
   - Comment and annotation features
   - Real-time collaboration

3. **Advanced Processing:**
   - Multi-language OCR support
   - Table extraction from documents
   - Document format conversion

4. **Integration:**
   - Microsoft Teams integration
   - Email integration for document submission
   - Third-party app connectors (Zapier)

---

## 12. Technical Stack Summary

**Frontend:**
- React 18+ or Angular 17+
- TypeScript
- Material-UI or Ant Design

**Backend:**
- ASP.NET Core 8 or Node.js 20
- Entity Framework Core or TypeORM
- RESTful API design

**Database:**
- Azure SQL Database or PostgreSQL

**Cloud Services:**
- Azure App Service
- Azure Functions
- Azure Blob Storage
- Azure AI Document Intelligence
- Azure Cognitive Search
- Azure AD B2C
- Application Insights

**DevOps:**
- Azure DevOps or GitHub Actions
- Infrastructure as Code (Bicep/Terraform)
- Automated CI/CD pipelines

---

## Conclusion

This architecture provides a robust, scalable, and cost-effective solution for the Document Hub project. The design leverages Azure's PaaS services to minimize infrastructure management while maintaining flexibility for future growth. The OCR integration using Azure AI Document Intelligence ensures high-quality text extraction, and Azure Cognitive Search provides powerful search capabilities across document content.

The phased implementation approach allows for early value delivery while managing risk and allowing for user feedback incorporation.

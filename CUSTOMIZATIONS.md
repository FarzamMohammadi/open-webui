# Customizations: Modifications from Upstream Open WebUI

## 2025-08-01: Shared Collections Implementation (Qdrant Only - commit 743e03645)
**Purpose**: Enable admins to upload documents that all users can access via RAG queries, creating a shared knowledge base alongside personal files.

**Architecture**: Elegant reuse of Qdrant's multitenancy system. Single unified collection stores all shared content with special tenant ID `__shared__`, automatically queried alongside user's personal files during RAG operations. Modified chat completion middleware to always trigger RAG pipeline when `SHARED_RAG_ENABLED=true`, ensuring shared knowledge is accessible in all conversations without requiring file attachments.

### **New API Endpoints**
- **`POST /files/shared`**: Admin-only endpoint for uploading shared documents
- **`GET /files/shared`**: List all shared files (accessible to all users)

### **Configuration Variables**
- **`SHARED_RAG_ENABLED`**: Feature toggle (default: false)
- **`SHARED_RAG_TENANT_ID`**: Special tenant ID for shared content (default: "__shared__")
- **`SHARED_RAG_COLLECTION_NAME`**: Collection name for shared documents (default: "shared_knowledge_base")

### **Technical Benefits**
- **Performance**: Single collection approach eliminates discovery overhead
- **Security**: Read-only access for users, admin-only uploads, proven tenant isolation
- **Maintainability**: Reuses existing patterns, comprehensive error handling, clean separation of concerns
- **Backwards Compatibility**: Feature toggle with validation, zero impact when disabled

### **Requirements**
- **VECTOR DB**: Qdrant only (feature specifically designed for Qdrant's multitenancy capabilities)
- **MANDATORY CONFIG**: Both `ENABLE_QDRANT_MULTITENANCY_MODE=true` AND `SHARED_RAG_ENABLED=true` must be set
- **ACCESS**: Admin role required for uploading shared documents
- **COMPATIBILITY**: Feature disabled by default, fully backward compatible

### **Usage Flow**
1. Admin enables feature + uploads docs via `POST /files/shared`
2. Documents stored with `__shared__` tenant ID in unified collection
3. All user RAG queries automatically include shared content alongside personal files (no file attachment needed)
4. Results clearly marked as originating from "Shared Knowledge Base"

```bash
# BOTH required - feature will not work without both settings
SHARED_RAG_ENABLED=true
ENABLE_QDRANT_MULTITENANCY_MODE=true
```

## 2025-07-18: Image Embedding for RAG
Images with OCR content now get embedded as files for RAG alongside vision model processing
- **Files Modified**: 1 file
  - `Chat.svelte`: Added `imageFilesWithContent` mapping logic to convert images to file format for RAG processing

## 2025-07-15: Image Content Extraction
- Backend: Enabled images in default processing pipeline (commit 743e03645)
  - Removed image exclusion from `process_file()` logic
  - Images now processed through content extraction engine
- Frontend: Images upload to backend for content extraction alongside vision processing (commit 697691efe)
  - Update `MessageInput.svelte` to show progress indicator for image uploads (commit d32d8d38a)
- Result: Images get OCR text extracted for RAG

## 2025-07-15: Docker Build
- Custom GitHub Actions workflow
- Multi-platform: AMD64 + ARM64
- Builds to `ghcr.io/farzammohammadi/open-webui:main`
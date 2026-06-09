# AnNa-ML: AI-Enhanced File Analysis Layer

**AI/ML satellite for AnNa file-sync platform.** Provides intelligent file analysis, categorization, and metadata extraction **without bloating** the core AnNa sync engine.

## Architecture Principle

**AnNa** (`AnNa`) = Peer-to-peer file synchronization (pure sync, zero AI)  
**AnNa-ML** (this repo) = Optional AI/ML analysis layer (pluggable, independent)

---

## Design Goals

- **Decoupled**: No dependencies on AnNa internals; communicates via REST API only
- **Optional**: AnNa works perfectly without this layer; ML features are add-ons
- **Standalone**: Runs as separate service; can be scaled independently
- **Specialized**: ML models and tensor operations stay here, not in AnNa

---

## Planned Components

### 1. File Classification Engine
```rust
// File type classification beyond MIME types
- Document classification (memo, report, contract, etc.)
- Code language detection + syntax awareness
- Image content recognition (photo, diagram, screenshot)
- Media type inference (video quality, audio codec)
```

### 2. Metadata Extraction
```rust
// Smart extraction from file content
- Document: Title, author, creation date, summary
- Code: Language, framework, dependencies, test coverage
- Images: Dimensions, EXIF data, text extraction (OCR)
- Archives: Contents inventory without extraction
```

### 3. Smart Deduplication
```rust
// Beyond hash-based dedup
- Similar file detection (fuzzy matching)
- Duplicate image detection (perceptual hashing)
- Text similarity for near-duplicates
```

### 4. Security Analysis
```rust
// Malware and threat assessment
- File signature validation
- Malware detection integration (ClamAV, VirusTotal)
- Suspicious pattern detection
```

### 5. Content Search & Indexing
```rust
// Full-text and semantic search
- OCR + text extraction indexing
- Image reverse search integration
- Semantic search via embeddings
```

---

## API Contract

### AnNa-ML Service Interface

```rust
// All communication via REST/HTTP, no shared data structures

POST /analyze
{
  "file_hash": "abc123...",
  "file_path": "path/to/file.pdf",
  "file_size": 1024000,
  "mime_type": "application/pdf"
}

Response:
{
  "classification": {
    "document_type": "report",
    "language": "english",
    "confidence": 0.95
  },
  "metadata": {
    "title": "Q3 Report",
    "author": "Team",
    "created": "2026-06-01"
  },
  "security": {
    "risk_level": "safe",
    "malware_scan": "clean"
  }
}
```

### Integration Points

**When a file arrives in AnNa:**
1. AnNa stores file normally (no changes)
2. AnNa optionally POSTs to AnNa-ML `/analyze` endpoint
3. AnNa-ML processes asynchronously, returns metadata
4. AnNa stores metadata in SQLite (optional enhancement)
5. User can query via AnNa API `/files/{hash}/metadata`

**No bloat in AnNa:** If AnNa-ML is not running, AnNa continues working perfectly.

---

## Technology Stack (Planned)

### Server
- **Rust + Tokio** — async processing
- **Axum** — REST API server
- **ONNX Runtime** — ML model execution (cross-platform)
- **Candle / ndarray** — tensor operations
- **ClamAV** — malware detection integration
- **Tesseract/OCR** — text extraction from images

### Models
- Vision Transformer (ViT) for image classification
- ELECTRA for document text classification
- SentenceTransformer for semantic search
- Perceptual hashing for duplicate detection

### Deployment
- Containerized (Docker) with model registry
- Optional: Model quantization for edge deployment
- GPU support where available, CPU fallback

---

## Repository Structure (Planned)

```
AnNa-ML/
├── server/
│   ├── src/
│   │   ├── main.rs
│   │   ├── api.rs              REST handlers
│   │   ├── classification.rs    File type classification
│   │   ├── metadata.rs         Metadata extraction
│   │   ├── security.rs         Malware detection
│   │   ├── indexing.rs         Search indexing
│   │   └── models.rs           Model loading + inference
│   ├── models/                 ONNX model files (git-lfs)
│   └── Cargo.toml
│
├── client/
│   ├── src/
│   │   └── lib.rs              Client SDK (optional)
│   └── package.json
│
├── tests/
│   └── integration.rs
│
└── docker/
    ├── Dockerfile              Standard + GPU variants
    └── models.env              Model registry URLs
```

---

## Security & Compliance

- **No data retention**: Analyzes files in memory, discards immediately
- **GDPR compliant**: No personal data stored or transmitted
- **Optional**: Users can run without ML layer for privacy-first sync
- **Sandbox**: Runs in separate container/process from AnNa

---

## Integration Example

```bash
# Start AnNa (pure file-sync)
cd AnNa && cargo run --release

# In another terminal, start AnNa-ML (optional)
cd AnNa-ML && cargo run --release

# AnNa automatically detects /analyze endpoint
# Files uploaded to AnNa are enriched with ML metadata
# If AnNa-ML is down, AnNa continues working normally
```

---

## Roadmap

### Phase 1: Foundation (Week 1-2)
- [x] Establish repo and architecture
- [ ] Basic REST API server
- [ ] Model loading pipeline

### Phase 2: Core Analysis (Week 3-4)
- [ ] Document classification
- [ ] Image classification
- [ ] Malware detection integration

### Phase 3: Advanced Features (Week 5-6)
- [ ] Semantic search indexing
- [ ] Similarity detection
- [ ] Custom model support

### Phase 4: Production Hardening (Week 7-8)
- [ ] Performance optimization
- [ ] GPU support
- [ ] Model quantization
- [ ] Monitoring & observability

---

## Design Decisions

### Q: Why separate from AnNa?
**A:** ML/tensor dependencies are heavy. AnNa users who want lightweight sync shouldn't pay the cost. This way, file-sync works offline and without Python/CUDA overhead.

### Q: How does AnNa know about AnNa-ML?
**A:** Service discovery: AnNa polls `/health` on `localhost:8001`. If healthy, it sends `POST /analyze` on each file. No hard coupling.

### Q: What if AnNa-ML crashes?
**A:** AnNa continues unaffected. Files sync normally, just without metadata. Users retry when ML service is healthy.

### Q: Can I run this on edge devices?
**A:** Yes. Quantized ONNX models fit on mobile/Raspberry Pi. AnNa-ML can be deployed to any Rust-capable device.

---

## Contributing

This is part of the JakeDot organization's security-first philosophy. All AI/ML features must:
- [ ] Not degrade AnNa's core file-sync performance
- [ ] Be independently deployable
- [ ] Have zero hard dependencies on AnNa internals
- [ ] Include security review (especially for model inputs)
- [ ] Respect user privacy (no data retention)

See `CONTRIBUTING.md` for guidelines.

---

**Status**: Planning phase  
**Sister Project**: [AnNa](https://github.com/JakeDot/AnNa)  
**Maintained by**: JakeDot Organization  

⚓

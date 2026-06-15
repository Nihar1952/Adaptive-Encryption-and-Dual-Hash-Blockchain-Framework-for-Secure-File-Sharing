# Decentralized File Auditing & Sharing System
## Technical Implementation Summary for IEEE Presentation

### 🔐 **Adaptive AES Encryption Logic**

**Key Implementation Details:**

- **Dynamic Algorithm Selection** based on sensitivity classification:
  - **LOW**: AES-128-GCM (128-bit key)
  - **MEDIUM**: AES-192-GCM (192-bit key)  
  - **HIGH**: AES-256-GCM (256-bit key)

- **Authenticated Encryption Approach**:
  - Uses AES-GCM (Galois/Counter Mode) for confidentiality AND authenticity
  - Auto-generates cryptographically secure random IV (Initialization Vector) per file
  - Extracts authentication tag post-encryption for integrity verification

- **Sensitivity-Driven Encryption Strength**:
  - ML module classifies file sensitivity at upload time
  - Key length adapts proportionally (128/192/256-bit)
  - Minimal overhead for low-sensitivity files; maximum security for high-sensitivity data

**Workflow:**
```
File Upload → ML Classification (Sensitivity) → Adaptive AES Selection → 
Encrypt with Random IV → Extract Auth Tag → Store Metadata
```

---

### 🔗 **Dual-Hash Verification Process**

**Cryptographic Hash Architecture:**

- **Plaintext Hash** (SHA-256):
  - Computed on original file before encryption
  - Enables integrity check against source file
  - Serves as unique file fingerprint

- **Ciphertext Hash** (SHA-256):
  - Computed on encrypted file after encryption
  - Validates integrity of encrypted data at rest
  - Detects tampering or corruption in storage

- **Hash Storage**:
  - Both hashes persisted in MongoDB alongside encrypted file metadata
  - Referenced during download/audit operations
  - Enables cryptographic proof of file provenance and immutability

**Verification Flow:**
```
Plaintext Hash ←→ Original File Integrity
Ciphertext Hash ←→ Encrypted Artifact Integrity (IPFS)
Both stored in DB for audit trail
```

---

### 📋 **Smart Contract Interactions for Metadata Storage**

**Blockchain Layer (Solidity Contract: FileAudit):**

- **Immutable Audit Records Structure**:
  - `fileId`: Unique file identifier
  - `userId`: Actor performing the action
  - `action`: Upload/Download event type
  - `cid`: IPFS Content Identifier (decentralized storage pointer)
  - `timestamp`: Block timestamp (tamper-proof)

- **Core Functions**:
  - `logAudit()`: Records file action events on-chain
  - `getRecord()`: Retrieves audit record by index
  - `getRecordCount()`: Total audit events count
  - Emits `AuditLogged` event for off-chain indexing

- **Decentralized Metadata Workflow**:
  - All file operations trigger blockchain logging
  - IPFS CID stored alongside on-chain audit entry
  - Creates immutable access history without storing file content
  - Enables forensic analysis and compliance auditing

**Integration Pattern:**
```
File Action (Upload/Download) 
    ↓
AES Encryption + Hash Generation (Backend)
    ↓
Upload to IPFS (Decentralized Storage)
    ↓
Log Audit Event to Blockchain (fileId, userId, action, CID)
    ↓
Immutable Record with Blockchain Timestamp
```

---

### 🏗️ **System Architecture Summary**

**Three-Layer Composition:**

1. **Storage Layer**: IPFS (distributed file storage)
2. **Metadata Layer**: MongoDB (hash records, encryption params, access control)
3. **Audit Layer**: Ethereum Smart Contract (immutable event log)

**Security Properties:**
- ✅ Confidentiality: Adaptive AES encryption
- ✅ Integrity: Dual-hash verification
- ✅ Authenticity: GCM authentication tags + blockchain timestamps
- ✅ Non-repudiation: Immutable on-chain audit trail

---

### 📊 **Recommended Slide Points for "Proposed Methodology"**

**Slide 1 - Encryption Layer:**
- Adaptive AES: 128/192/256-bit based on ML-detected sensitivity
- GCM mode provides both encryption + authentication
- Random IV per file; auth tag validates ciphertext integrity

**Slide 2 - Hashing & Verification:**
- Dual-hash strategy: plaintext (before) + ciphertext (after)
- SHA-256 for both; enables end-to-end integrity validation
- Hashes stored in MongoDB for quick retrieval & audit queries

**Slide 3 - Blockchain Integration:**
- Solidity smart contract logs all file operations immutably
- Stores: fileId, userId, action, IPFS-CID, on-chain timestamp
- Decentralized audit trail without storing file content on-chain
- Enables forensic analysis & regulatory compliance

**Slide 4 - System Overview Diagram:**
```
┌─────────────────────────────────────────────────────────┐
│            User Upload/Download                         │
└────────────────┬────────────────────────────────────────┘
                 │
        ┌────────▼────────┐
        │  ML Classifier  │ → Determine Sensitivity
        └────────┬────────┘
                 │
    ┌────────────┴─────────────────┐
    │                              │
┌───▼────────────┐    ┌──────────▼────────┐
│ Adaptive AES   │    │  Hash Generation  │
│ Encryption     │    │  (Plaintext +     │
│                │    │   Ciphertext)     │
└───┬────────────┘    └──────────┬────────┘
    │                            │
    ├────────────────┬───────────┤
    │                │           │
┌───▼────────┐ ┌────▼──────┐ ┌─▼──────────┐
│   IPFS     │ │ MongoDB   │ │ Blockchain │
│ (Storage)  │ │ (Metadata)│ │ (Audit)    │
└────────────┘ └───────────┘ └────────────┘
```


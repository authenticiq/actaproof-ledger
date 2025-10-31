# ActaProof Public Ledger

**Immutable, publicly verifiable ledger of ActaProof merkle tree anchors**

This repository contains the daily merkle tree roots for all receipts created on the ActaProof platform. Each commit represents a cryptographic anchor point that proves when receipts were created and ensures they cannot be backdated or tampered with.

## 🔐 What is This?

ActaProof creates **receipts** for links and files—think of them as cryptographic timestamps proving "this content existed at this time." To prevent anyone (including us) from backdating or altering receipts, we:

1. **Batch receipts daily** at 2 AM UTC
2. **Build a merkle tree** from all receipts created that day
3. **Publish the root hash** to two independent locations:
   - **Supabase Storage** (primary storage with public access)
   - **GitHub** (this repository, for distributed verification)

## 📁 Repository Structure

```
/
├── daily/
│   ├── 2025-10-01.json   # Merkle root + proofs for Oct 1, 2025
│   ├── 2025-10-02.json   # Merkle root + proofs for Oct 2, 2025
│   └── ...
├── index.json             # List of all published dates
├── latest.json            # Most recent merkle root
└── README.md              # This file
```

## 🔍 Verifying a Receipt

Each ActaProof receipt includes:
- `anchor_root`: The merkle root hash for its batch date
- `anchor_proof`: An array of hashes proving the receipt is in the tree
- `anchor_chain`: Set to `"actaproof-log"`

### Manual Verification Steps:

1. **Find the daily file** for the receipt's creation date
2. **Fetch the file** from this repository: `daily/YYYY-MM-DD.json`
3. **Verify the merkle root** matches `anchor_root` in the receipt
4. **Verify the proof path** by hashing up the tree using `anchor_proof`
5. **Check Git commit timestamp** to confirm publication time

### Automated Verification:

```bash
# Example: Verify receipt against GitHub ledger
curl https://raw.githubusercontent.com/authenticiq/actaproof-ledger/main/daily/2025-10-31.json \
  | jq '.root'
# Compare with receipt's anchor_root field
```

## 📊 Daily File Format

Each `daily/YYYY-MM-DD.json` file contains:

```json
{
  "date": "2025-10-31",
  "algorithm": "sha256",
  "leaves_format": "sha256:<hex>",
  "count": 47,
  "root": "abc123...",
  "duplicates_dropped": 0,
  "published_to": ["supabase-storage", "github-commits"],
  "proofs": {
    "sha256:def456...": ["abc...", "def...", "ghi..."]
  }
}
```

**Fields:**
- `date`: Batch date in YYYY-MM-DD format
- `algorithm`: Hash algorithm used (sha256)
- `leaves_format`: Format of leaf hashes
- `count`: Number of receipts in this batch
- `root`: Merkle root hash
- `duplicates_dropped`: Number of duplicate receipts excluded
- `published_to`: Where this batch was published
- `proofs`: Map of leaf hash → merkle proof path

## 🛡️ Trust Properties

### Why GitHub?

1. **Git Commit History**: GitHub timestamps every commit—backdating requires rewriting Git history, which is publicly visible
2. **Distributed Trust**: Anyone can clone this repo and verify the entire history
3. **Third-Party Infrastructure**: GitHub's infrastructure provides external validation beyond our own systems
4. **Free & Public**: No API keys required, fully open for verification
5. **Redundancy**: If Supabase Storage fails, GitHub provides backup

### What This Proves:

✅ **Receipts existed at claimed time** (Git commit timestamp)  
✅ **Cannot be backdated** (Git history immutable without detection)  
✅ **Cannot be altered** (Changing a receipt changes merkle root, breaks proofs)  
✅ **Publicly auditable** (Anyone can clone and verify)

### What This Doesn't Prove:

❌ **Not Bitcoin-level immutability** (GitHub could theoretically rewrite history)  
❌ **Not distributed consensus** (Single point of trust: our automation)  
❌ **Not zero-trust** (You trust GitHub's infrastructure + our honest automation)

## 🔗 Additional Verification

For maximum trust, ActaProof receipts can also be verified against:

1. **Supabase Storage**: Cross-check files at `${LEDGER_BASE_URL}/daily/YYYY-MM-DD.json`
2. **API Endpoint**: `GET /api/v1/ledger/root?date=YYYY-MM-DD`
3. **Database Records**: Query `anchor_batches` table in Supabase
4. **Receipt Signatures**: Each receipt has ML-DSA-65 quantum-resistant signature
5. **JWKS Public Keys**: Verify signatures against public key set

## 🚀 Future Enhancements

Planned additions:
- **Bitcoin Anchoring** (OpenTimestamps): Anchor merkle roots to Bitcoin blockchain
- **Signed Commits**: Sign Git commits with ML-DSA-65 quantum-resistant signatures
- **IPFS Mirroring**: Additional distributed storage layer
- **Blockchain Explorer**: Web UI for browsing and verifying anchors

## 📝 License

Public domain. This data is intended for public verification and audit.

## 🤝 Contact

- **Website**: https://actaproof.com
- **API Docs**: https://api.actaproof.com/reference
- **Issues**: https://github.com/authenticiq/actaproof-ledger/issues

---

**Automated updates**: This repository is updated daily at ~2:05 AM UTC by the ActaProof anchoring service.

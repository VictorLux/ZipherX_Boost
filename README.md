# ZipherX Unified Boost File

A single, comprehensive blockchain data file for instant ZipherX wallet synchronization.

## Overview

| Property | Value |
|----------|-------|
| **Format** | ZBOOST01 (Unified Binary) |
| **Version** | 1 |
| **Chain Height** | 2,935,268 |
| **File Size** | 747.5 MB (uncompressed) |
| **Created** | 2025-12-07 |

## What's Inside?

The unified boost file contains **all data** needed for fast wallet synchronization in a single download:

| Section | Count | Description |
|---------|-------|-------------|
| **Shielded Outputs** | 1,042,536 | Encrypted notes for trial decryption |
| **Shielded Spends** | 431,880 | Nullifiers for spent note detection |
| **Block Hashes** | 2,458,300 | For P2P header validation (Sapling onwards) |
| **Block Timestamps** | 2,458,300 | For transaction date display |
| **Serialized Tree** | 478 bytes | Commitment tree state for instant load |
| **Reliable Peers** | 9 | P2P bootstrap addresses |

## File Format Specification

### Header (128 bytes)

| Offset | Size | Field | Description |
|--------|------|-------|-------------|
| 0 | 8 | Magic | `ZBOOST01` (ASCII) |
| 8 | 4 | Version | Format version (1) |
| 12 | 8 | Chain Height | End block height (uint64 LE) |
| 20 | 8 | Sapling Height | Sapling activation (476,969) |
| 28 | 32 | Tree Root | Commitment tree root hash |
| 60 | 32 | Block Hash | Hash at chain height |
| 92 | 4 | Section Count | Number of sections |
| 96 | 8 | Created At | Unix timestamp |
| 104 | 24 | Reserved | Padding for future use |

### Section Types

| ID | Name | Record Size | Description |
|----|------|-------------|-------------|
| 1 | Outputs | 652 bytes | height(4) + index(4) + cmu(32) + epk(32) + ciphertext(580) |
| 2 | Spends | 36 bytes | height(4) + nullifier(32) |
| 3 | Hashes | 32 bytes | Block hash in wire format (little-endian) |
| 4 | Timestamps | 4 bytes | Unix timestamp (uint32 LE) |
| 5 | Tree | Variable | Serialized Sapling commitment tree |
| 6 | Peers | Variable | Peer addresses |

### Section Data Layout

```
[Header: 128 bytes]
[Outputs Data: 1,042,536 × 652 = ~679.7 MB]
[Spends Data: 431,880 × 36 = ~15.5 MB]
[Hashes Data: 2,458,300 × 32 = ~78.7 MB]
[Timestamps Data: 2,458,300 × 4 = ~9.8 MB]
[Tree Data: 478 bytes]
[Peers Data: 195 bytes]
```

**Total: ~747.5 MB**

## Byte Order Convention

All multi-byte integers are **little-endian** (matching wire format):

- CMUs: Little-endian (wire format, NOT display format)
- EPKs: Little-endian (wire format)
- Nullifiers: Little-endian (wire format)
- Block hashes: Little-endian (wire format, reversed from RPC display)
- Integers: Little-endian (uint16, uint32, uint64)

**Important**: RPC/API responses display hashes in big-endian. The file stores them in wire format (bytes reversed).

## Height Ranges

| Section | Start Height | End Height | Notes |
|---------|--------------|------------|-------|
| Outputs | 476,969 | 2,935,268 | From Sapling activation |
| Spends | 476,969 | 2,935,268 | From Sapling activation |
| Hashes | 476,969 | 2,935,268 | From Sapling (no pre-Sapling hashes) |
| Timestamps | 476,969 | 2,935,268 | From Sapling activation |
| Tree | 476,969 | 2,935,268 | Sapling commitment tree |

## Verification

```bash
# Verify SHA256 checksum
shasum -a 256 -c SHA256SUMS.txt

# Or manually
shasum -a 256 zipherx_boost_v1.bin
# Expected: f1e94bf4f13d0c8633887c1e2d02ab7f6c09afe6621c463e239c00785ea02c12
```

## Usage

The ZipherX wallet automatically:

1. Checks for updates on GitHub releases
2. Downloads the unified file if newer version available
3. Parses sections on-demand during sync
4. Uses bundled data for instant wallet initialization

### For Imported Wallets

When importing a private key, the wallet:

1. Downloads the unified boost file (~747 MB)
2. Uses parallel note decryption (Rayon) for fast scanning
3. Computes nullifiers to detect spent notes
4. Builds witnesses for spendable notes

### For New Wallets

New wallets skip historical note scanning since there are no notes to find - only recent blocks are synced.

## Technical Details

### Cryptographic Values

| Property | Value |
|----------|-------|
| Sapling Activation | 476,969 |
| Chain Height | 2,935,268 |
| Block Hash | `000003d0100e65b486e3af98e9825edd808641824ca64bb7a72ab6d3d8e43636` |
| Tree Root | `42911deac4be9e18489faecea30c05f116b6a89f868e5aa3502311fb1ef6f630` |

### Shielded Output Record (652 bytes)

```
struct ShieldedOutput {
    height: u32,           // 4 bytes - Block height
    index: u32,            // 4 bytes - Output index
    cmu: [u8; 32],         // 32 bytes - Note commitment (wire format)
    epk: [u8; 32],         // 32 bytes - Ephemeral public key (wire format)
    ciphertext: [u8; 580], // 580 bytes - Encrypted note
}
```

### Shielded Spend Record (36 bytes)

```
struct ShieldedSpend {
    height: u32,           // 4 bytes - Block height
    nullifier: [u8; 32],   // 32 bytes - Nullifier (wire format)
}
```

## Generation Statistics

| Metric | Value |
|--------|-------|
| Generation Speed | 2,629 blocks/sec |
| Total Blocks Scanned | 2,458,300 |
| Generation Time | 16.8 minutes |
| RPC Batch Size | 200 blocks |
| Worker Threads | 48 |

## GitHub Release

The unified boost file is distributed via GitHub Releases:

- **Repository**: VictorLux/ZipherX_Boost
- **Release Tag**: v2935268-unified
- **Files**: zipherx_boost_v1.bin, zipherx_boost_manifest.json, SHA256SUMS.txt

---

*ZipherX - Privacy is a right, not a privilege.*

# ZipherX Unified Boost File

A single, comprehensive blockchain data file for instant ZipherX wallet synchronization.

## Overview

| Property | Value |
|----------|-------|
| **Format** | ZBOOST01 (Unified Binary) |
| **Version** | 1 |
| **Chain Height** | 2,955,907 |
| **File Size** | 1093.1 MB (uncompressed) |
| **Created** | 2025-12-25 |

## What's Inside?

The unified boost file contains **all data** needed for fast wallet synchronization in a single download:

| Section | Count | Description |
|---------|-------|-------------|
| **Shielded Outputs** | 1,043,674 | Encrypted notes for trial decryption |
| **Shielded Spends** | 432,962 | Nullifiers for spent note detection |
| **Block Hashes** | 2,478,939 | For P2P header validation (Sapling onwards) |
| **Block Timestamps** | 2,478,939 | For transaction date display |
| **Serialized Tree** | 478 bytes | Commitment tree state for instant load |
| **Reliable Peers** | 9 | P2P bootstrap addresses |
| **Block Headers** | 2,475,168 | FIX #413: Full headers with finalSaplingRoot for Tree Root Validation |

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
| 7 | Headers | 140 bytes | FIX #413: version(4) + prevHash(32) + merkleRoot(32) + saplingRoot(32) + time(4) + bits(4) + nonce(32) |

### Section Data Layout

```
[Header: 128 bytes]
[Outputs Data: 1,043,674 × 652 = ~649.0 MB]
[Spends Data: 432,962 × 36 = ~14.9 MB]
[Hashes Data: 2,478,939 × 32 = ~75.7 MB]
[Timestamps Data: 2,478,939 × 4 = ~9.5 MB]
[Tree Data: 478 bytes]
[Peers Data: 195 bytes]
[Headers Data: 2,475,168 × 140 = ~330.3 MB] (FIX #413)
```

**Total: ~1,092.2 MB** (with headers section)

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
| Outputs | 476,969 | 2,955,907 | From Sapling activation |
| Spends | 476,969 | 2,955,907 | From Sapling activation |
| Hashes | 476,969 | 2,955,907 | From Sapling (no pre-Sapling hashes) |
| Timestamps | 476,969 | 2,955,907 | From Sapling activation |
| Tree | 476,969 | 2,955,907 | Sapling commitment tree |

## Verification

```bash
# Verify SHA256 checksum
shasum -a 256 -c SHA256SUMS.txt

# Or manually
shasum -a 256 zipherx_boost_v1.bin
# Expected: 1f72ca053f0acf0fae24205d9ef618ad8e2f0435152b59852a8aa8898428f533
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
| Chain Height | 2,955,907 |
| Block Hash | `00000669b3f66b90dc4d50c0a3f5279c691b491ff06284a6dec9dff6c896f4c2` |
| Tree Root | `5c39c9f04b24fc1f5b8d22b2e36542853be9a8587365b5e022c0dfc2c08b5fca` |

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
| Generation Speed | 2,185 blocks/sec |
| Total Blocks Scanned | 2,478,939 |
| Generation Time | 18.9 minutes |
| RPC Batch Size | 200 blocks |
| Worker Threads | 48 |

## GitHub Release

The unified boost file is distributed via GitHub Releases:

- **Repository**: VictorLux/ZipherX_Boost
- **Release Tag**: v2935268-unified
- **Files**: zipherx_boost_v1.bin, zipherx_boost_manifest.json, SHA256SUMS.txt

---

*ZipherX - Privacy is a right, not a privilege.*

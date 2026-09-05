# Forensic Data Carving & Data-Unit File Recovery

This repository contains the laboratory deliverables, evidence verification records, and complete forensic examination report for Module 7: Data Carving and File Recovery (CIP-B102).

The investigation evaluates and compares filesystem-aware recovery techniques against raw signature-based data carving on damaged or deleted file systems.

---

## 📋 Executive Overview

* **Examiner:** Nebeuwa Ifeanyichukwu Raphael (Junior Digital Forensic Analyst)
* **Platform:** Kali Linux (`kali@kali`)
* **Core Toolkit:** `xxd`, `exiftool`, `strings`, `The Sleuth Kit (TSK)` (`img_stat`, `mmls`, `fsstat`, `fls`, `icat`, `blkcat`), `binwalk`, `dd`, `Scalpel`, `7z`, `sha256sum`, `md5sum`
* **Evidence Examined:**
  * `J_ub_law.jpg`: JPEG sample image for byte-level signature and EXIF/XMP metadata analysis.
  * `Ch01InChap01.dd`: Raw 1.44MB FAT12 floppy disk image for filesystem and data-unit inspection.
  * `120M.7z` / `usb_fat_carving.001`: Raw 124MB FAT16 USB drive image for partition verification and signature carving.

---

## 🛠️ Key Findings & Methodology

### 1. Hexadecimal Analysis & Lossless Reconstruction (`xxd`)
* Verified JPEG Start-of-Image (`FF D8 FF`) and End-of-Image (`FF D9`) signatures on `J_ub_law.jpg`.
* Dumped plain hex via `xxd -p` and performed reverse-hexdump reconstruction using `xxd -r -p`.
* **Integrity Check:** Reconstructed file hashes matched the original file byte-for-byte (`SHA-256: 238ff34393c50e52c0e8b14fcff8ec7dc29e23914dbc435f8ef998d172a91468`).

### 2. Embedded EXIF/XMP Metadata Analysis (`exiftool`)
* Extracted provenance data showing the image is a heavily reprocessed derivative (15 recorded Photoshop/Camera Raw edits between 2014 and 2020).
* Original RAW filename discovered: `UB_10-8-13_cam2_0117.jpg` (Original camera: **Nikon D4**).

### 3. Filesystem & Inode-Level Recovery (The Sleuth Kit)
* Analyzed `Ch01InChap01.dd` (FAT12):
  * `mmls`: Confirmed superfloppy structure (no partition table).
  * `fls`: Identified deleted directory entries (`r/r *`).
  * `icat`: Recovered deleted text files (`recovered_confirmation.txt`, `recovered_billing_letter.doc`) directly by inode numbers (11, 8).
  * `blkcat`: Extracted raw Cluster 33 data, uncovering a Microsoft Access Database header (`Standard Jet DB`).

### 4. Manual Embedded Artifact Carving (`dd` & `binwalk`)
* `binwalk` flagged top-level TIFF/EXIF structures but did not carve embedded thumbnail streams automatically.
* Using offset/length parameters (`skip=1198`, `count=4934`) derived from metadata, `dd` was used to manually carve an embedded 160×90 JPEG thumbnail (`extracted_thumbnail.jpg`), visually verified as intact.

### 5. USB Image Partition Offset Verification (`mmls` / `file`)
* Analyzed `usb_fat_carving.001`: Independent examination via `mmls` and `file` confirmed sectors 0–127 are unallocated MBR code, with the Win95 FAT16 filesystem strictly beginning at **Sector 128** (Offset: 65,536 bytes).

### 6. Signature-Based File Carving (`Scalpel`)
* Configured `/etc/scalpel/scalpel.conf` for JPEG header/footer patterns (`Exif` and `JFIF`).
* Executed Scalpel across `usb_fat_carving.001`, carving **17 JPEG files**.
* Two carving results (`00000000.jpg`, `00000001.jpg`) were visually verified as uncorrupted image files.
* **Forensic Observation:** No hash match was observed between the carved files and the original sample `J_ub_law.jpg`. This highlights the primary limitation of raw signature carving: non-contiguous cluster fragmentation breaks contiguous header-to-footer carves, whereas filesystem-aware tools (`icat`) succeed by following FAT cluster chains.

---

## 📊 Cryptographic Verification Ledger

| Artifact / Evidence File | MD5 Hash | SHA-256 Hash |
| :--- | :--- | :--- |
| `Ch01InChap01.dd` | `a117773bcf1fc88ec0ab8e0a349fbbcb` | `3ce8053e4f3d9c8ab98b3aadb2480685efb8e4980d34297b83bd5a09b1a7b122` |
| `J_ub_law.jpg` | `83a360ac7f7e0ca318e5bfe39f95f137` | `238ff34393c50e52c0e8b14fcff8ec7dc29e23914dbc435f8ef998d172a91468` |
| `120M.7z` | `dfe7b5424e54cd1bf50d5df47aceeb3c` | `2d2a3d93c9ec65bcad9f89c9894429ea72caa8add07726a3b96ec9a6ab6a58ce` |
| `usb_fat_carving.001` | `ba4a1d0ba49f4a6667b00a3b3e85e604` | `9bfe4b5634ade30764f0e581a4686930f7fab0472378595b789b0cdb248c91d9` |
| `recovered_confirmation.txt` | — | `51b981e09b72739fac67c2d0124a2603cae8c35e075f5e2f66fbad03d53afcc0` |
| `extracted_thumbnail.jpg` | — | `f47359e0aca4b23c876ac78888819e0c5a58e71e53c0a1a0f9005115a20c4c9d` |

---

## 📂 Repository Structure

```text
.
├── CIP-B102_Lab3_Data Carving with XXD, Binwalk and Scalpel.pdf  # Full formal lab report
├── README.md                                                     # Project documentation
├── evidence/                                                     # Raw evidence media (unmodified)
├── hex_analysis/                                                 # Plain hex dumps & reconstructed JPEGs
├── tsk_output/                                                   # Files & blocks recovered via TSK (icat/blkcat)
├── binwalk_output/                                               # Embedded content & dd manually carved artifacts
└── scalpel_output/                                               # Scalpel carve results & audit.txt log

<h1 align="center">🔍 The Wiped USB That Told On Itself</h1>
<p align="center"><em>A 24-hour digital forensics CTF — recovering evidence from a deliberately destroyed FAT16 drive</em></p>

<p align="center">
<img src="https://img.shields.io/badge/Status-Completed-brightgreen">
<img src="https://img.shields.io/badge/Platform-Linux-blue">
<img src="https://img.shields.io/badge/Digital-Forensics-red">
<img src="https://img.shields.io/badge/FileSystem-FAT16-orange">
<img src="https://img.shields.io/badge/License-Educational-lightgrey">
</p>

<p align="center">
<b>Case Reference:</b> CARTEL-USB-2026-08 &nbsp;·&nbsp;
<b>Examiner:</b> Obi David Chibuzor &nbsp;·&nbsp;
<b>Date:</b> 3 August 2026
</p>

---

## 📌 TL;DR

- A USB seized by EFCC officers, claimed by its owner to hold **"only personal photographs."**
- The only files actually visible on the drive? **Two gumbo recipes.**
- **80% of the drive** had been deliberately overwritten with a repeating "SORRY" / "CHARLIE" pattern — a textbook anti-forensic wipe.
- Carving unallocated space recovered **10 files that survived the wipe** — including a Word document that is, essentially, **the suspect's own diary entry confessing to the wipe**, written moments before it happened.
- One passage mentions hiding "rhino photos." The recovered files include **actual rhino images.** The diary isn't speculation — it's ground truth.
- No content directly tying the device to drug trafficking was found — the wipe may have taken it with it.

> **This report draws no conclusion on guilt or innocence — only on what the bytes actually show.**

---

## 🗂️ Repository Map

```
cartel_forensics/
├── README.md
├── evidence/cartel.rar        📦 original disk image, compressed (1.7 MB)
├── recovered/                 🧩 every file carved from the wipe, untouched
│   ├── 00104057.jpg  00104249.jpg  00105065.jpg   (alligators)
│   ├── 00105873.jpg  00335081.jpg                 (alligator — duplicate pair)
│   ├── 00106393.jpg  00106409.jpg                 (rhinos)
│   ├── 00106865.gif  00106889.gif                 (rhino art)
│   ├── 00335017.doc                               (the diary)
│   └── audit.txt                                  (foremost carving log)
└── images/                    🖼️ process screenshots referenced below
```

---

## 🧭 How This Investigation Unfolded

<details>
<summary><b>Step 1 — Lock the evidence down before touching anything</b></summary>

<br>

Before a single command touched the image, it was hashed. This is the difference between "trust me" and "verify me" — anyone can re-hash `evidence/cartel.rar` right now and confirm it's the exact image examined here.

| Algorithm | Value |
|---|---|
| MD5 | `80348c58eec4c328ef1f7709adc56a54` |
| SHA-256 | `ce550424200a997c61b413941c8ef4df9619a2f96579674952294a176a32be65` |

<p align="center"><img src="images/fig1-1-hash-verification.png" width="560"></p>

</details>

<details>
<summary><b>Step 2 — Figure out what this drive even is</b></summary>

<br>

Raw image, no partition table, FAT16, 247.5 MB, formatted with Linux `mkdosfs`. A small, unremarkable USB stick — on paper.

| Property | Value |
|---|---|
| Format | Raw (dd-style) |
| File system | FAT16, no MBR (superfloppy) |
| Size | 259,506,176 bytes |
| Volume serial | `0x4092D9D1` |

<p align="center">
<img src="images/fig2-1-mmls-output.png" width="270">
<img src="images/fig2-3-fsstat-output.png" width="270">
</p>

</details>

<details>
<summary><b>Step 3 — Check what the suspect's claim would predict we'd find</b></summary>

<br>

If the claim ("only personal photographs") were true, the file listing should show photos. It shows two text files.

**`GUMBO1.TXT`** and **`GUMBO2.TXT`** — recipes, written 2004-04-30, and the *only* two allocated files on the entire volume. That's it. That's everything a normal user would see if they plugged this drive in.

<p align="center"><img src="images/fig3-1-fls-listing.png" width="560"></p>

**Claim vs. reality, one line in:** ❌

</details>

<details>
<summary><b>Step 4 — Notice the drive is mostly... text?</b></summary>

<br>

Scanning unallocated space turned up something odd: huge stretches of the disk weren't random garbage, weren't zeros, and weren't leftover file fragments. They were **the same word, repeated, over and over, for megabytes.**

- `SORRY` repeated **8,835,577 times** — ~54.5 MB
- `CHARLIE` repeated **18,372,608 times** — ~148.5 MB

Combined: **208 MB. Over 80% of the entire drive**, deliberately overwritten in two distinct passes.

<p align="center">
<img src="images/fig3-4-sorry-pattern.png" width="270">
<img src="images/fig3-5-charlie-pattern.png" width="270">
</p>

This isn't what a quick-format or accidental deletion looks like. Ordinary deletion leaves the old bytes sitting in unallocated space, fully recoverable. **This was wiped on purpose.**

</details>

<details open>
<summary><b>Step 5 — Find what survived the wipe anyway</b></summary>

<br>

Between the tiny sliver of allocated space and the start of the wipe, a ~1.5 MB "island" of untouched data survived. Foremost carved **6 JPEGs and 2 GIFs** out of it.

<p align="center">
<img src="recovered/00104057.jpg" height="150">
<img src="recovered/00104249.jpg" height="150">
<img src="recovered/00105065.jpg" height="150">
<img src="recovered/00105873.jpg" height="150">
</p>
<p align="center"><em>Alligators. Stock photography — one still carries a "copyright 2000 philg@mit.edu" comment baked in.</em></p>

<p align="center">
<img src="recovered/00106393.jpg" height="150">
<img src="recovered/00106409.jpg" height="150">
<img src="recovered/00106865.gif" height="130">
<img src="recovered/00106889.gif" height="130">
</p>
<p align="center"><em>Rhinos. Keep this in mind — it matters a lot in Step 6.</em></p>

None of these carry EXIF, GPS, or camera metadata. They're not personal photographs. They're internet imagery.

</details>

<details open>
<summary><b>Step 6 — Recover the file that changes everything</b></summary>

<br>

Deep inside the CHARLIE-wiped region, an 11 MB Word document survived — because `foremost.conf` has the `.doc` carving rule disabled by default, it was missed on the first pass. Re-enabling it and re-running the carve is what actually found it.

What's inside isn't a report. It's a **diary entry**, written in the first person, dated 9 August 2005:

> *"...I zapped the hard drive and then threw it into the Mississippi River. I'm gonna reformat my USB key after this entry, but try not to destroy the good stuff..."*

That sentence **is the wipe from Step 4**, described by its own author, in the past-about-to-become-present tense.

> *"Rhino pictures illegal? Makes me sick. I 'hid' the photos..."*

That sentence **is Step 5**. The rhino images aren't a coincidence — they're a direct, confirmed match to what the diary says was hidden.

<p align="center">
<img src="images/fig4-1b-doc-text-drive.png" width="270">
<img src="images/fig4-1c-doc-text-rhino.png" width="270">
</p>

This is the rare case where the anti-forensic actor left behind their own signed confession, sitting right next to the crime scene.

</details>

<details>
<summary><b>Step 7 — Catch the file that was there twice</b></summary>

<br>

Hashing every recovered file turned up a coincidence that isn't one: `00105873.jpg` (pre-wipe island) and `00335081.jpg` (buried 117 MB away, inside the CHARLIE region) are **byte-for-byte identical** — re-verified here directly against the actual files in `recovered/`.

```
SHA-256:  f92654d9ee17ab6b684b09de01cf0bc4076383c007964946d3f31577447596fb
```

Same alligator-hatching photo, present on both sides of the wipe boundary — a small but useful anchor point for sequencing what happened when.

</details>

<details>
<summary><b>Step 8 — Piece together a timeline, with the gaps left honest</b></summary>

<br>

<p align="center"><img src="images/fig-timeline-diagram.png" width="560"></p>

Most of this timeline is *relative*, not absolute — the wipe destroyed the metadata that would normally date everything. Only two hard timestamps survive on the entire drive, and they're from **2004** — over 20 years before the 2026 seizure, almost certainly an unset system clock rather than a real date.

| # | Event | When |
|---|---|---|
| 1–2 | Gumbo recipes written | 2004-04-30 (hard timestamp) |
| 3 | Photos written | Before the wipe (inferred) |
| 4 | First wipe — SORRY | Undated |
| 5 | Diary + duplicate photo present | Undated |
| 6 | Second wipe — CHARLIE | Undated |
| 7 | Device seized | 2026-08-02 (chain of custody) |

</details>

---

## 🎯 Mapped to MITRE ATT&CK®

| What happened | Technique |
|---|---|
| The SORRY/CHARLIE wipe | [`T1485` Data Destruction](https://attack.mitre.org/techniques/T1485/) |
| The diary's own account of the wipe | [`T1485` Data Destruction](https://attack.mitre.org/techniques/T1485/) *(corroborating)* |
| Zero surviving directory entries for any carved file | [`T1070.004` Indicator Removal — File Deletion](https://attack.mitre.org/techniques/T1070/004/) |

---

## ⚠️ What Actually Matters Here (Risk Snapshot)

| Finding | Confidence | Why it matters |
|---|---|---|
| 🔴 Deliberate wipe, >80% of drive | High | Establishes anti-forensic intent, full stop |
| 🔴 Diary confirms the wipe *and* the hidden photos | High | Self-corroborating — rare in casework |
| 🟠 Claim vs. reality mismatch (recipes ≠ photos) | Confirmed | Undermines the suspect's stated claim directly |
| 🟡 Duplicate file across the wipe boundary | Confirmed | Useful for timeline, not standalone evidence |
| ⚪ No drug-trafficking content recovered | N/A | Absence of evidence ≠ evidence of absence, given the wipe |

---

## 🧩 What Got in the Way

- `foremost.conf` ships with `.doc` carving **disabled by default** — the single most important file in this case was almost missed entirely because of a default config setting.
- Telling a *deliberate* wipe apart from an *accidental* one required scanning the full image for pattern repetition, not just sampling a few offsets — sampling alone would have looked like "weird garbage data," not "intentional destruction."
- The diary and the Word document metadata were originally documented in two different places before being merged into one coherent narrative here.
- Until the actual image files were reviewed, the "rhino photos" line in the diary was just a thematic guess. Having the real files turned it into a confirmed match.

---

## 🎓 Skills This Case Demonstrates

`NIST SP 800-86 methodology` `Evidence integrity (MD5/SHA-256)` `The Sleuth Kit` `Foremost & Binwalk carving` `Anti-forensic pattern detection` `Cross-artefact correlation` `MITRE ATT&CK mapping` `Constrained timeline reconstruction`

---

## 🛠️ Tools

`mmls` `fsstat` `fls` `istat` `icat` `blkls` · `foremost` · `binwalk` · `exiftool` · `catdoc` · `md5sum` / `sha256sum` · Python 3

---

## 📚 Reference

- [The Sleuth Kit](https://www.sleuthkit.org/) · [Foremost](http://foremost.sourceforge.net/) · [Binwalk](https://github.com/ReFirmLabs/binwalk)
- NIST SP 800-86 — Guide to Integrating Forensic Techniques into Incident Response
- [MITRE ATT&CK®](https://attack.mitre.org/) — `T1485`, `T1070.004`
- `recovered/audit.txt` — the original foremost carving log, included as-is

<p align="center"><sub>This report documents technical findings only. It does not assert guilt or innocence.</sub></p>

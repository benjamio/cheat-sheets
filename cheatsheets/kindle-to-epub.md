# Kindle to EPUB/PDF Conversion Cheat Sheet

**Purpose:** Convert DRM-protected Kindle books to EPUB/PDF for use on non-Kindle devices (e.g., reMarkable).  
**Assumptions:** Windows x86 machine (or VM). Kindle book purchased legitimately. ARM/Windows on ARM may have compatibility issues with Kindle for PC and the decryption tools.  
**Last updated:** 2026-04-08

---

## Why this is needed

- Kindle books use KFX format with DRM that requires Windows-specific tools to decrypt.
- The Kindle desktop app on macOS/Linux cannot be used for this workflow.
- Kindle for PC on Windows x86 is currently the most reliable free path.

---

## Quick overview

```
Kindle for PC (Windows x86)
  → KFXArchiver283 (decrypt)
    → Calibre + KFX Input plugin (convert)
      → EPUB or PDF
```

---

## Step 1: Install Kindle for PC

1. On your Windows x86 machine, install **Kindle for PC** (version 2.8.x).
2. Sign in to your Amazon account and download the book.

---

## Step 2: Install Visual C++ Redistributable

KFXArchiver283 requires the Microsoft Visual C++ runtime:

1. Download **vc_redist.x64.exe** from Microsoft.
2. Install it.

---

## Step 3: Download Satsuoni's DeDRM tools

1. Go to `github.com/Satsuoni/DeDRM_tools/releases`.
2. Download the latest release (v10.0.18 as of April 2026).
3. Extract the zip.

---

## Step 4: Decrypt with KFXArchiver283

1. **Keep Kindle for PC open** with the book downloaded.
2. Create an `output` folder next to the executable.
3. Run:

```cmd
KFXArchiver283.exe "C:\Users\<USER>\Documents\My Kindle Content" "C:\Users\<USER>\Desktop\output" "C:\Users\<USER>\Desktop\output\key.k4i"
```

- First argument: Kindle content folder (contains `_EBOK` subfolders)
- Second argument: output folder (must exist)
- Third argument: path for the extracted key file

This produces decrypted `.kfx-zip` files in the output folder.

---

## Step 5: Convert with Calibre

1. Install **Calibre** (`calibre-ebook.com`).
2. Install the **KFX Input** plugin:
   - Preferences → Plugins → Get new plugins → search "KFX Input" → Install.
3. Drag the decrypted `.kfx-zip` into Calibre.
4. Right-click the book → **Convert books** → choose **EPUB** or **PDF**.
5. Find the output: right-click → **Open containing folder**.

Default output location: `C:\Users\<USER>\Calibre Library\<Book Title>\`

---

## EPUB vs PDF

| Format | Best for |
|--------|----------|
| EPUB | Reflowable text, comfortable reading, reMarkable (reflows natively) |
| PDF | Fixed layout, technical books with diagrams, annotation with fixed positioning |

**Recommendation:** Use EPUB unless you need fixed-layout diagrams.

---

## Troubleshooting

- **"MSVCP140.dll was not found"** — Install Visual C++ Redistributable (Step 2).
- **"Usage: executable ..."** — You need all three arguments: content path, output folder, key file path.
- **"Output folder must exist"** — Create the output folder before running.
- **ARM/Windows on ARM compatibility issues** — Kindle for PC and the decryption tools may not work on ARM processors; use an x86 machine or x86 VM.
- **KFX Input plugin not recognizing file** — Ensure you're importing the `.kfx-zip`, not the raw folder.

---

## Tool versions (verified April 2026)

| Tool | Version |
|------|---------|
| Kindle for PC | 2.8.x (2.8.70995) |
| Satsuoni DeDRM | v10.0.18 |
| KFX Input (Calibre plugin) | 2.30.0 |
| Calibre | Latest stable |

---

## References

- Satsuoni DeDRM fork: `github.com/Satsuoni/DeDRM_tools`
- KFX Input plugin: MobileRead Forums (`mobileread.com/forums/showthread.php?t=291290`)
- Calibre: `calibre-ebook.com`

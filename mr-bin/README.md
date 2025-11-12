# Write-up — *Mr.Bin* (CTFlearn)

**Category:** Steganography / Digital Forensics
**Flag:** `CTFlearn{y0u_n4il3d_it}`

---

## TL;DR

The challenge file was a regular JPEG that secretly had an encrypted ZIP file appended to it. After carving out the ZIP, finding the password inside the image, and extracting a long binary text file, I rebuilt the data into a 600×600 PBM image, converted it to PNG, fixed the orientation, and revealed the flag.

---

## Environment & Tools

* OS: Kali Linux
* Tools: `file`, `binwalk`, `dd`, `strings`, `7z`, `tr`, `fold`, `convert` (ImageMagick)

---

## Step 1 — Inspecting the File

I started with the given file `image.jpg` and ran a few quick checks:

```bash
file image.jpg
identify -format "%wx%h %m\n" image.jpg
```

Both commands confirmed it was a normal JPEG. To look for anything hidden inside, I used `binwalk`:

```bash
binwalk -e image.jpg
```

`binwalk` detected a ZIP archive appended to the end of the file at an offset (`53488` bytes in my case). The ZIP was encrypted.

---

## Step 2 — Carving the Hidden ZIP

Using the offset reported by `binwalk`, I carved out the appended data into a new file:

```bash
dd if=image.jpg of=hidden.zip bs=1 skip=53488 status=progress
file hidden.zip
```

The output confirmed `hidden.zip` was indeed a valid ZIP archive, but it was password-protected.

---

## Step 3 — Finding the Password

I looked for any readable text or hidden clues inside both files:

```bash
strings image.jpg | head -n 200
strings hidden.zip | grep -i pass -n || true
```

One of the strings revealed a clue: `pass: mr_b1n`. That had to be the ZIP password.

---

## Step 4 — Extracting the ZIP

Using the password, I extracted the ZIP archive:

```bash
7z x -pmr_b1n hidden.zip -ohidden_mrbin
# or alternatively:
unzip -P mr_b1n hidden.zip -d hidden_unzip
```

Inside, there was a file named `bin` that contained a very long string of `0` and `1` characters.

---

## Step 5 — Understanding the Data

Looking back at the original challenge image, I found a Base64 string `NjAweDYwMF9waWN0dXJl` hidden in the metadata. Decoding it gave `600x600_picture`, which suggested the image size for reconstructing the binary file.

---

## Step 6 — Rebuilding the Image

I created a PBM image (plain text bitmap) using the binary data:

```bash
printf "P1\n600 600\n" > img.pbm
tr -cd '01' < hidden_mrbin/bin | fold -w 600 >> img.pbm
```

The PBM format expects `P1` on the first line, width/height on the second, and rows of 0s and 1s (where `0` = white and `1` = black).

---

## Step 7 — Converting & Viewing

Then I converted it to PNG:

```bash
convert img.pbm img.png
display img.png
```

The image showed text, but it was rotated and mirrored.

---

## Step 8 — Fixing the Orientation

A simple rotate and flop command corrected the view:

```bash
convert img.png -rotate 90 -flop fixed.png
display fixed.png
```

And there it was — the clear text flag:

```
CTFlearn{y0u_n4il3d_it}
```

---

## Why This Works

* `binwalk` detects appended or embedded files.
* `dd` carves binary data precisely by offset.
* `strings` often reveals readable hints or hidden passwords.
* The bitstream was raw pixel data; knowing its dimensions lets us rebuild it into an actual image.
* The orientation fix is a common last step when reconstructing raw binary images.

---

## Key Takeaways

* Always check for **appended data** in image challenges.
* Base64 strings in metadata or comments often contain vital hints.
* PBM (Portable Bitmap) format is perfect for reconstructing `0`/`1` image data.
* Rotate/flip adjustments can reveal the flag when the raw bit order is reversed.

---

## Final Flag

```
CTFlearn{y0u_n4il3d_it}
```

---

This was a fun, clean stego/forensics challenge. It perfectly combined file carving, metadata analysis, and basic image reconstruction — all core skills in digital forensics CTFs.

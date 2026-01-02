# Write-up — unopenable.gif (CTF / Digital Forensics)

This document presents a complete forensic analysis of the **unopenable.gif** challenge.
The objective was not only to recover the flag, but to understand why the file failed to open,
verify its internal structure, and restore it using minimal, forensic-safe modifications.

---

## TL;DR
The challenge involved a GIF file that could not be opened due to missing header bytes.
After restoring the correct GIF header and analyzing the animation frames, the file was found
to contain Base64-encoded fragments distributed across multiple frames.
Combining and decoding these fragments revealed the final flag:

**flag{g1f_or_j1m}**

---

## Environment & Tools
- OS: Kali Linux
- Tools:
  - `file`
  - `hexdump`
  - `dd`
  - `convert` (ImageMagick)
  - `gifsicle`
  - `base64`

---

## Initial Assessment
The problem was approached as a **file format integrity issue**, rather than a rendering or viewer problem.
The first step was to determine whether the file was still identifiable at the binary level.

~~~bash
file unopenable.gif
~~~

Output:
~~~
data
~~~

This result indicates that the file’s magic bytes or header were missing or corrupted.

---

## Header Inspection
To confirm this, the beginning of the file was examined:

~~~bash
hexdump -C unopenable.gif | head -n 10
~~~

The output began with:
~~~
39 61 f4 01 f4 01 ...
~~~

A valid GIF file must begin with one of the following headers:
- `GIF87a`
- `GIF89a`

In this case, only the final two bytes of the header (`39 61` → `9a`) were present,
while the first four bytes were missing.
This confirmed that the file was still structurally a GIF, but its identifying header had been partially removed.

---

## Restoring the GIF Header
To preserve forensic integrity, all modifications were performed on a copy of the original file:

~~~bash
cp unopenable.gif fixed.gif
~~~

Because the file was later confirmed to be animated, the `GIF89a` header was restored:

~~~bash
printf '\x47\x49\x46\x38\x39\x61' | dd of=fixed.gif bs=1 conv=notrunc
~~~

Verification after restoration:

~~~bash
file fixed.gif
~~~

Result:
~~~
GIF image data, 500 x 500
~~~

This confirmed that the file format had been successfully repaired.

---

## Frame Analysis
Opening the repaired file revealed an **animated GIF** consisting of five frames:

1. A fully black frame (introductory frame)
2. A frame containing a Base64 text fragment
3. A frame containing a Base64 text fragment
4. A frame containing a Base64 text fragment
5. A final frame displaying the text **“decode it”**

The final frame served as an explicit instruction for the next step.

---

## Extracting Frames
Each frame was extracted for individual analysis:

~~~bash
mkdir frames
convert fixed.gif frames/frame-%02d.png
~~~

Extracted content:
- `frame-00.png` → black frame
- `frame-01.png` → `ZmxhZ3tn`
- `frame-02.png` → `MWZfb3`
- `frame-03.png` → `JfajFmfQ==`
- `frame-04.png` → `decode it`

---

## Decoding the Hidden Data
The three Base64 fragments were concatenated in order:

~~~
ZmxhZ3tnMWZfb3JfajFmfQ==
~~~

The combined string was decoded using:

~~~bash
echo 'ZmxhZ3tnMWZfb3JfajFmfQ==' | base64 -d
~~~

Output:
~~~
flag{g1f_or_j1m}
~~~

---

## Key Takeaways
- File header validation should be an early step in file-based forensic analysis
- Magic bytes are critical for file type identification
- Minimal, controlled modification using `dd` is sufficient to restore damaged headers
- Animated GIFs can distribute hidden data across multiple frames
- Base64 encoding is commonly used to conceal readable data in CTF challenges

---

## Command Summary
~~~bash
cp unopenable.gif fixed.gif
file unopenable.gif
hexdump -C unopenable.gif | head -n 10
printf '\x47\x49\x46\x38\x39\x61' | dd of=fixed.gif bs=1 conv=notrunc
convert fixed.gif frames/frame-%02d.png
echo 'ZmxhZ3tnMWZfb3JfajFmfQ==' | base64 -d
~~~

---

## Final Flag
~~~
flag{g1f_or_j1m}
~~~

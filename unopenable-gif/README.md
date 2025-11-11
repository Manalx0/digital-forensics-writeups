# Write-up — unopenable.gif (CTF/DF)

## TL;DR

The challenge started with a broken GIF file that wouldn’t open because its header was missing. After fixing the header and exploring the frames, the GIF turned out to have five frames: one black frame, three frames with Base64-encoded text, and a final frame saying “decode it.” When the three text fragments were combined and decoded, they revealed the final flag:

**`flag{g1f_or_j1m}`**

---

## Environment & Tools

* OS: Kali Linux
* Tools used: `file`, `hexdump`, `dd`, `convert`, `gifsicle`, `base64`

---

## Step 1: First Look

When I first ran `file` on `unopenable.gif`, it just said **data** — meaning the header was corrupted or missing.

So I dumped the first bytes:

```bash
hexdump -C unopenable.gif | head -n 10
```

Instead of seeing `GIF89a`, the first bytes were:

```
39 61 f4 01 f4 01 ...
```

Only the last two bytes (`39 61` → `9a`) from the header were left. The first four were gone.

At that point, I suspected the file was still a GIF — just with its magic bytes removed.

---

## Step 2: Fixing the Header

I created a safe copy to work on:

```bash
cp unopenable.gif fixed.gif
```

Then wrote the correct GIF header (`GIF89a`) to the beginning:

```bash
printf '\x47\x49\x46\x38\x39\x61' | dd of=fixed.gif bs=1 conv=notrunc
```

After checking again:

```bash
file fixed.gif
```

It finally showed **GIF image data, 500 x 500**, confirming the structure was intact.

---

## Step 3: Opening and Exploring

Opening the file revealed an **animated GIF** with five frames:

1. A completely black frame (blank intro)
2. A frame with Base64 text fragment #1
3. A frame with Base64 text fragment #2
4. A frame with Base64 text fragment #3
5. A final frame that says **“decode it”**

The design clearly hinted at the next step — combine and decode the text.

---

## Step 4: Extracting the Frames

I extracted each frame using ImageMagick:

```bash
mkdir frames && convert fixed.gif frames/frame-%02d.png
```

After extraction, I had:

```
frame-00.png  (black)
frame-01.png  (ZmxhZ3tn)
frame-02.png  (MWZfb3)
frame-03.png  (JfajFmfQ==)
frame-04.png  (decode it)
```

---

## Step 5: Combining and Decoding

The three meaningful frames contained:

```
ZmxhZ3tn , MWZfb3 , JfajFmfQ==
```

I removed the commas/spaces and joined them:

```
ZmxhZ3tnMWZfb3JfajFmfQ==
```

Then decoded it:

```bash
echo 'ZmxhZ3tnMWZfb3JfajFmfQ==' | base64 -d
```

Output:

```
flag{g1f_or_j1m}
```

The fifth frame’s message — “decode it” — was literally the instruction for this step.

---

## Step 6: What I Learned

* Always check the **header**; a missing header is a common beginner CTF trick.
* **GIF headers** start with `GIF87a` or `GIF89a`.
* A single `dd` command can restore missing magic bytes.
* Animated GIFs can hide multi-part clues in separate frames.
* Base64 encoding often hides readable text in plain sight.

---

## Step 7: Useful Commands Summary

```bash
cp unopenable.gif work-unopenable.gif
file work-unopenable.gif
hexdump -C work-unopenable.gif | head -n 10
printf '\x47\x49\x46\x38\x39\x61' | dd of=fixed.gif bs=1 conv=notrunc
convert fixed.gif frames/frame-%02d.png
echo 'ZmxhZ3tnMWZfb3JfajFmfQ==' | base64 -d
```

---

## Final Flag

```
flag{g1f_or_j1m}
```

---

This challenge was a great intro to digital forensics concepts — simple but clever. It taught me how missing headers can be repaired, how to inspect GIF structures, and how even basic encoding like Base64 can be used creatively to hide a flag.

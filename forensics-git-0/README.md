# Forensics Git 0 — Write-up

## Overview

This challenge provided a compressed disk image. The objective was to recover the flag from within the image using a forensic approach rather than simply browsing files.

---

## Step 1: File Identification and Extraction

I began by identifying the file type and extracting it:

```bash
file disk.img.gz
gzip -d disk.img.gz
file disk.img
```

The output showed that the file is a **full disk image with an MBR partition table**, not a standalone filesystem. This means I cannot directly list files without first identifying the correct partition.

---

## Step 2: Partition Analysis

To understand the disk structure, I used:

```bash
fdisk -l disk.img
mmls disk.img
```

From `mmls`, I observed three partitions:

- Partition 1: Linux  
- Partition 2: Swap  
- Partition 3: Linux  

The swap partition was not useful for structured file analysis, so I focused on the Linux partitions.  

The key observation was:

```
Start sector: 1140736
```

This indicates where the third partition begins inside the disk image.

---

## Step 3: Accessing the Filesystem via Offset

Instead of mounting the image, I used Sleuth Kit tools to avoid modifying the evidence.

```bash
fls -o 1140736 disk.img
```

This command lists the root directory of the filesystem by starting at the correct offset.

---

## Step 4: Navigating Using Inodes

Rather than using file paths, I followed inode-based navigation.

```bash
fls -o 1140736 disk.img
```

```
d/d 64770: home
```

```bash
fls -o 1140736 disk.img 64770
```

```
d/d 64771: ctf-player
```

```bash
fls -o 1140736 disk.img 64771
```

```
d/d 65663: Code
```

```bash
fls -o 1140736 disk.img 65663
```

```
d/d 65664: secrets
```

```bash
fls -o 1140736 disk.img 65664
```

```
d/d 65665: .git
r/r 65692: note.txt
```

At this point, the directory name **"secrets"** stood out as suspicious, so I prioritized investigating it.

---

## Step 5: Inspecting note.txt

```bash
icat -o 1140736 disk.img 65692
```

Output:

```
The picoCTF flag format is 'picoCTF{}'
```

This confirmed the expected flag format but did not reveal the flag itself.

---

## Step 6: Investigating Git Artifacts

The presence of a `.git` directory was significant, especially given the challenge name.

```bash
fls -o 1140736 disk.img 65665
```

```
r/r 65693: COMMIT_EDITMSG
d/d 65703: logs
```

Git directories often contain historical or hidden data, so I focused on them.

---

## Step 7: Extracting the Commit Message

```bash
icat -o 1140736 disk.img 65693
```

Output:

```
Wrap this phrase in the flag format: g17_1n_7h3_d15k_041217d8
```

This was clearly the hidden phrase needed for the flag.

---

## Final Flag

```
picoCTF{g17_1n_7h3_d15k_041217d8}
```

---

## Key Takeaways

- A disk image must be treated as a full disk, not a direct filesystem.
- `mmls` is essential for identifying partition boundaries.
- `fls` allows navigation using inode values instead of file paths.
- `icat` retrieves file contents directly from disk structures.
- Git metadata can contain valuable forensic artifacts.

---

## Analyst Notes

Initially, I considered mounting the partition for easier access. However, I chose to use Sleuth Kit tools instead to avoid altering any potential evidence and to maintain a forensic workflow.

Rather than scanning all files blindly, I followed a structured approach:
- Identify the correct partition
- Focus on user directories
- Look for anomalies or suspicious naming
- Investigate metadata (in this case, Git)

This approach significantly reduced unnecessary effort and led directly to the flag.

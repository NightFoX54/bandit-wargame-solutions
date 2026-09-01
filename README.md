# OverTheWire: Bandit Wargame Solutions & Linux CLI Notes
 
This repository contains my write-ups, terminal commands, and text-processing notes for the **OverTheWire Bandit** wargame. It serves as a personal reference guide for Linux CLI fundamentals, shell navigation, text manipulation, archive handling, and networking.
 
---
 
## 🛠 Tech Stack & Tools Used
 
* **File Operations:** `cat`, `ls`, `cd`, `find`, `file`
* **Text Processing:** `grep`, `sort`, `uniq`, `strings`, `base64`, `tr`
* **Data & Archives:** `xxd`, `gzip`, `bzip2`, `tar`
* **Networking & SSH:** `ssh`, `scp`, `nc` (netcat)
---
 
## 🚀 Level Walkthroughs
 
### Level 0 ➔ Level 1
 
* **Concept:** Establishing basic SSH connection and reading standard text files.
* **Key Commands:** `ssh`, `cat`
```bash
cat readme
```
 
### Level 1 ➔ Level 2
 
* **Concept:** Reading files with special characters as filenames (dashed filename `-`).
* **Key Commands:** `cat`
```bash
cat ./-
```
 
### Level 2 ➔ Level 3
 
* **Concept:** Handling filenames with spaces using single quotes or escaping.
* **Key Commands:** `cat`
```bash
cat ./'--spaces in this filename--'
```
 
### Level 3 ➔ Level 4
 
* **Concept:** Viewing hidden files (files starting with `.`).
* **Key Commands:** `ls -la`, `cat`
```bash
cd inhere
ls -la
cat ./...Hiding-From-You
```
 
### Level 4 ➔ Level 5
 
* **Concept:** Identifying human-readable (ASCII) files among binary data using wildcards and `file`.
* **Key Commands:** `file`, `grep`
```bash
file ./* | grep ASCII
cat ./-file07
```
 
### Level 5 ➔ Level 6
 
* **Concept:** Advanced file searching using specific criteria (file size, permissions, type).
* **Key Commands:** `find`, `cat`
```bash
find . -type f -size 1033c ! -executable
cat ./maybehere07/.file2
```
 
### Level 6 ➔ Level 7
 
* **Concept:** System-wide search filtering by user ownership, group ownership, and redirecting error streams (`2>/dev/null`).
* **Key Commands:** `find`, `cat`
```bash
find / -size 33c -group bandit6 -user bandit7 2>/dev/null
cat /var/lib/dpkg/info/bandit7.password
```
 
### Level 7 ➔ Level 8
 
* **Concept:** Filtering specific terms from large text files.
* **Key Commands:** `grep`
```bash
grep millionth data.txt
```
 
### Level 8 ➔ Level 9
 
* **Concept:** Finding unique lines. Requiring sorting prior to `uniq` since it only detects adjacent duplicate lines.
* **Key Commands:** `sort`, `uniq`
```bash
sort data.txt | uniq -u
```
 
### Level 9 ➔ Level 10
 
* **Concept:** Extracting human-readable strings from binary data.
* **Key Commands:** `strings`, `grep`
```bash
strings data.txt | grep "==="
```
 
### Level 10 ➔ Level 11
 
* **Concept:** Decoding Base64 encoded data.
* **Key Commands:** `base64`
```bash
base64 -d data.txt
```
 
### Level 11 ➔ Level 12
 
* **Concept:** Decrypting ROT13 substitution ciphers using character mapping.
* **Key Commands:** `tr`
```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```
 
### Level 12 ➔ Level 13
 
* **Concept:** Reversing hexdumps and handling recursively compressed multi-layered archives (bzip2, gzip, tar).
* **Key Commands:** `xxd`, `file`, `mv`, `gzip`, `bzip2`, `tar`
```bash
# Revert hex dump to binary
xxd -r data.txt out1
 
# Iteratively inspect file type, append proper extension, and decompress
file out1
mv out1 out1.gz && gzip -d out1.gz
mv out1 out1.bz2 && bzip2 -d out1.bz2
mv out1 out1.tar && tar -xvf out1.tar
```
 
### Level 13 ➔ Level 14
 
* **Concept:** SSH key-based authentication and transferring files across networks using `scp`.
* **Key Commands:** `scp`, `ssh -i`
```bash
# Transfer private key to local machine
scp -P 2220 bandit13@bandit.labs.overthewire.org:~/sshkey.private .
 
# Connect using private key
ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
```
 
### Level 14 ➔ Level 15
 
* **Concept:** Submitting credentials to a network service listening on a local port.
* **Key Commands:** `nc` (netcat)
```bash
nc localhost 30000
```

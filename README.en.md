# OverTheWire: Bandit Wargame Solutions & Linux CLI Notes

This repository contains my write-ups, terminal commands, and text-processing techniques for the **OverTheWire Bandit** wargame. It was put together as a personal reference guide covering Linux command-line fundamentals, shell navigation, text manipulation, archive/compression handling, networking, SSH, cron jobs, and Git.

---

## 🛠 Tools Used

* **File Operations:** `cat`, `ls`, `cd`, `find`, `file`, `chmod`
* **Text Processing:** `grep`, `sort`, `uniq`, `strings`, `base64`, `tr`, `diff`
* **Archives & Compression:** `xxd`, `gzip`, `bzip2`, `tar`
* **Networking & SSH:** `ssh`, `scp`, `nc` (netcat), `openssl s_client`
* **Version Control:** `git` (clone, log, branch, tag, push)
* **Other:** cron job analysis, setuid binaries, restricted shell escape techniques

---

## 📚 Table of Contents

| Level | Topic |
|---|---|
| 0 → 1 | Connecting via SSH, `cat` |
| 1 → 2 | Filenames starting with a dash (`-`) |
| 2 → 3 | Filenames containing spaces |
| 3 → 4 | Hidden files |
| 4 → 5 | Identifying file type with `file` |
| 5 → 6 | Filtering by size/permissions with `find` |
| 6 → 7 | Filtering by ownership with `find`, redirecting error streams |
| 7 → 8 | Searching for a word with `grep` |
| 8 → 9 | Finding a unique line with `sort` + `uniq -u` |
| 9 → 10 | Extracting readable strings with `strings` |
| 10 → 11 | Decoding Base64 |
| 11 → 12 | Decrypting ROT13 |
| 12 → 13 | Reversing a hexdump + unwrapping a multi-layer archive |
| 13 → 14 | Authenticating with an SSH key, `scp` |
| 14 → 15 | Sending a password over a port with `nc` |
| 15 → 16 | Connecting to an SSL/TLS port with `openssl s_client` |
| 16 → 17 | Scanning a port range and capturing an SSH private key |
| 17 → 18 | Finding the difference between two files with `diff` |
| 18 → 19 | Running a single command over SSH in a restricted `.bashrc` environment |
| 19 → 20 | Running a command as another user via a setuid binary |
| 20 → 21 | Setting up an `nc` listener for client-server communication |
| 21 → 22 | A cron job leaking a world-readable temp file |
| 22 → 23 | Tracking a filename computed at runtime with `md5sum` in a cron job |
| 23 → 24 | Dropping your own script into a folder a cron job executes |
| 24 → 25 | Brute-forcing a PIN code with `nc` |
| 25 → 26 | Escaping a restricted shell via `more`/`vim` |
| 26 → 27 | Continuing with a setuid binary |
| 27 → 28 | Cloning a repo with `git clone` |
| 28 → 29 | Finding deleted data in commit history with `git log -p` |
| 29 → 30 | A hidden branch found via `git branch -a` and `git checkout` |
| 30 → 31 | A hidden tag found via `git tag` and `git show` |
| 31 → 32 | Bypassing `.gitignore` with `git add -f` and pushing |
| 32 → 33 | Escaping an uppercase-only shell with `>>` |
| 33 | The final level of the game |

---

## 🚀 Level Walkthroughs

### Level 0 ➔ Level 1

* **Topic:** Establishing a basic SSH connection and reading plain text files.
* **What I learned:** Connect to the server with `ssh -p <port> user@host`, list files in the home directory with `ls`, and read their contents with `cat`.

```bash
ssh -p 2220 bandit0@bandit.labs.overthewire.org
ls
cat readme
```

### Level 1 ➔ Level 2

* **Topic:** Reading a file whose name is just `-` (a dash).
* **What I learned:** A command like `cat -` interprets `-` as a request to read from stdin. Prefixing the path with `./` avoids this ambiguity.

```bash
cat ./-
```

### Level 2 ➔ Level 3

* **Topic:** Reading filenames that contain spaces.
* **What I learned:** Filenames with spaces can be wrapped in single quotes (`'...'`) or escaped with `\` so the shell treats them as one argument.

```bash
cat ./'--spaces in this filename--'
```

### Level 3 ➔ Level 4

* **Topic:** Viewing hidden files (those starting with a dot).
* **What I learned:** `ls` doesn't show hidden files by default; the `-a` (all) flag is needed.

```bash
cd inhere
ls -la
cat ./...Hiding-From-You
```

### Level 4 ➔ Level 5

* **Topic:** Finding the one human-readable (ASCII) file among many in a folder.
* **What I learned:** `file` guesses a file's type (data, ASCII text, OpenPGP key, etc.). Running it against a wildcard (`./*`) and piping to `grep` narrows down the candidates.

```bash
file ./* | grep ASCII
cat ./-file07
```

### Level 5 ➔ Level 6

* **Topic:** Searching for a file matching specific size and permission criteria.
* **What I learned:** `find` can filter by size (`-size`), type (`-type f`), and permission (`! -executable`, i.e. "NOT executable").

```bash
find . -type f -size 1033c ! -executable
cat ./maybehere07/.file2
```

### Level 6 ➔ Level 7

* **Topic:** Searching the entire filesystem by ownership (user/group) and suppressing error output.
* **What I learned:** `find /` searches from the root directory; `-group` and `-user` filter by ownership. `2>/dev/null` suppresses stderr noise such as permission-denied errors.

```bash
find / -size 33c -group bandit6 -user bandit7 2>/dev/null
cat /var/lib/dpkg/info/bandit7.password
```

### Level 7 ➔ Level 8

* **Topic:** Searching for a specific word inside a large text file.
* **What I learned:** `grep <word> <file>` finds line-based matches.

```bash
grep millionth data.txt
```

### Level 8 ➔ Level 9

* **Topic:** Finding the one line in a file that appears exactly once.
* **What I learned:** `uniq -u` only removes duplicates that are **adjacent**, so the file must be sorted first so identical lines end up next to each other.

```bash
sort data.txt | uniq -u
```

### Level 9 ➔ Level 10

* **Topic:** Extracting readable text from a file that's mostly binary data.
* **What I learned:** `strings` lists printable character sequences found in a file; the output can be narrowed with `grep` for a distinctive marker (here, `=`).

```bash
strings data.txt | grep =
```

### Level 10 ➔ Level 11

* **Topic:** Decoding Base64-encoded data.
* **What I learned:** `base64 -d` (decode) turns Base64 content back into plain text.

```bash
base64 -d data.txt
```

### Level 11 ➔ Level 12

* **Topic:** Decrypting a ROT13 (letter-shift) cipher.
* **What I learned:** `tr` performs a one-to-one mapping between character sets; ROT13 decoding shifts letters by 13 positions (`A-Za-z` → `N-ZA-Mn-za-m`).

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

### Level 12 ➔ Level 13

* **Topic:** Converting a hexdump back to binary, then unwrapping a nested, multi-layer archive (a mix of gzip/bzip2/tar).
* **What I learned:** `xxd -r` reverts a hex dump to its original binary form. From there, each layer is identified with `file` (or `file -Z` to see inside compressed data as well), renamed (`mv`) with the correct extension, and extracted with the matching tool (`gzip -d`, `bzip2 -d`, `tar -xvf`). Since the number of remaining layers is unknown up front, this becomes a repeating **identify → rename → extract** loop.

```bash
# Convert the hex dump back to binary
xxd -r data.txt out1

# To peek inside compressed data too:
file -Z out1

# At each layer: identify the type, give it the right extension, extract
mv out1 out1.gz  && gzip  -d out1.gz
mv out1 out1.bz2 && bzip2 -d out1.bz2
mv out1 out1.tar && tar   -xvf out1.tar
# ... repeat until the file shows up as "ASCII text"
cat out_final
```

### Level 13 ➔ Level 14

* **Topic:** Authenticating with an SSH private key instead of a password, and transferring files over the network.
* **What I learned:** `scp` downloads a file from the remote server to the local machine; `ssh -i <key>` then uses that private key instead of a password to connect.

```bash
scp -P 2220 bandit13@bandit.labs.overthewire.org:~/sshkey.private .
ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220
```

### Level 14 ➔ Level 15

* **Topic:** Submitting credentials to a network service listening on a local port.
* **What I learned:** `nc <host> <port>` connects to a TCP port and sends data (here, the current password) over stdin; once the service receives the correct data, it returns the next level's password.

```bash
nc localhost 30000
```

### Level 15 ➔ Level 16

* **Topic:** Connecting to a port encrypted with SSL/TLS instead of a plain one.
* **What I learned:** If the target port expects SSL, a plain `nc` connection won't work — `openssl s_client -connect <host>:<port>` is needed instead.

```bash
openssl s_client -connect localhost:30001
```

### Level 16 ➔ Level 17

* **Topic:** Scanning a range of ports to find the right SSL port, then capturing an SSH private key in return.
* **What I learned:** To see which ports are open and what service each is running across a wide range, scan with `nmap`; `-p` sets the port range and `-sV` enables service/version detection, which reveals which port is expecting SSL. The `-quiet` flag simplifies `openssl s_client`'s output. Connecting to the right port and sending the current password causes the server to return the next level's SSH private key, which is saved to a file with corrected permissions (`chmod 600`) and used to log in directly.

```bash
# Scan the port range to see what's running where
nmap -p 31000-32000 -sV localhost

openssl s_client -connect localhost:31790 -quiet
# ... send the password, save the private key returned in response
```

### Level 17 ➔ Level 18

* **Topic:** Detecting the difference between two similar files.
* **What I learned:** `diff file1 file2` shows line-by-line differences — here used to find the single changed line between an old and new password list.

```bash
diff passwords.old passwords.new
```

### Level 18 ➔ Level 19

* **Topic:** Running a command in a restricted user environment whose `.bashrc` automatically logs the session out (`exit`) right after login.
* **What I learned:** Instead of opening an interactive shell, passing the command to run directly as an argument to `ssh` lets it execute before the automatic exit in `.bashrc` kicks in.

```bash
ssh -p 2220 bandit18@bandit.labs.overthewire.org "cat readme"
```

### Level 19 ➔ Level 20

* **Topic:** Running a command as another user via a SUID (setuid) binary.
* **What I learned:** The `s` bit in permissions like `-rwsr-x---` means the program runs with the privileges of its owner (here, `bandit20`). Running such a binary allows reading files that would otherwise be inaccessible.

```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

### Level 20 ➔ Level 21

* **Topic:** Opening a listener on a local port and having a client program connect to it.
* **What I learned:** `nc -l -p <port>` starts a background listener that serves up the current password; the target program (`./suconnect <port>`) then connects to that port, verifies the password it receives, and sends back the next one.

```bash
echo "<current_password>" | nc -l -p 4444 &
./suconnect 4444
```

### Level 21 ➔ Level 22

* **Topic:** Inspecting scheduled tasks (cron jobs) defined under `/etc/cron.d/`.
* **What I learned:** A cron job was copying the next level's password into a temporary (`/tmp`) file and making it world-readable (`chmod 644`); that file could then be read directly.

```bash
ls /etc/cron.d/
cat /etc/cron.d/cronjob_bandit22
cat /usr/bin/cronjob_bandit22.sh
cat /tmp/<file_created_by_the_script>
```

### Level 22 ➔ Level 23

* **Topic:** Precomputing a filename that a cron script generates dynamically at runtime using `md5sum`.
* **What I learned:** The script derived its target filename with `echo "I am user $myname" | md5sum`. Running the same command manually predicts the filename in advance so it can be read directly.

```bash
ls /etc/cron.d/
cat /etc/cron.d/cronjob_bandit23
cat /usr/bin/cronjob_bandit23.sh

mytarget=$(echo I am user bandit23 | md5sum | cut -d ' ' -f 1)
cat /tmp/$mytarget
```

### Level 23 ➔ Level 24

* **Topic:** Noticing that a cron job periodically runs (and then deletes) scripts in a specific folder (`/var/spool/<user>/foo`) if they're owned by the right user, and dropping your own script there.
* **What I learned:** The cron script only executed a file if its owner (`owner`) matched the expected user. A script written under our own account (bandit23) — one that reads the next level's password into a world-readable file — was copied into the target folder so cron would run it automatically.

```bash
ls /etc/cron.d/
cat /etc/cron.d/cronjob_bandit24
cat /usr/bin/cronjob_bandit24.sh

mkdir /tmp/workdir && chmod 777 /tmp/workdir
cd /tmp/workdir
nano answer.sh
# contents of answer.sh:
#   #!/bin/bash
#   cat /etc/bandit_pass/bandit24 > /tmp/workdir/answer
#   chmod 777 /tmp/workdir/answer
chmod 777 answer.sh
cp ./answer.sh /var/spool/bandit24/foo
# once cron runs it:
cat answer
```

### Level 24 ➔ Level 25

* **Topic:** Brute-forcing a 4-digit PIN by trying every possibility against a network service.
* **What I learned:** A bash range expression like `{0000..9999}` generates every PIN combination in a single loop, sent one by one to the service via `nc`; once the correct PIN is found, the service returns the next password.

```bash
for pin in {0000..9999}; do
    echo "<current_password> $pin";
done | nc localhost 30002
```

### Level 25 ➔ Level 26

* **Topic:** Escaping to a normal shell from an account that automatically launches (and closes) a restricted program on login, by exploiting terminal size and `more`/`vim` behavior.
* **What I learned:** Logging in with the SSH key caused a long text to be paged through `more`. Shrinking the terminal window triggered the `--More--` prompt, pressing `v` dropped into the Vim editor, and from Vim's command mode `:set shell=/bin/bash` followed by `:shell` produced a full bash shell.

```bash
scp -P 2220 bandit25@bandit.labs.overthewire.org:~/bandit26.sshkey .
ssh -i ./bandit26.sshkey bandit26@bandit.labs.overthewire.org -p 2220
# at the "--More--" prompt:  v
# in Vim's command line:
:set shell=/bin/bash
:shell
cat /etc/bandit_pass/bandit26
```

### Level 26 ➔ Level 27

* **Topic:** Again reading a file with another user's privileges via a SUID binary.
* **What I learned:** A continuation of the same idea from Level 19 — an executable owned by a different user runs whatever commands it allows with that user's privileges.

```bash
./bandit27-do cat /etc/bandit_pass/bandit27
```

### Level 27 ➔ Level 28

* **Topic:** Cloning a Git repository over SSH.
* **What I learned:** `git clone ssh://<user>@<host>:<port>/<repo-path>` downloads a remote Git repository over the SSH protocol; the files inside are then inspected normally.

```bash
git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
cd repo
cat readme
```

### Level 28 ➔ Level 29

* **Topic:** Information redacted from the current file still being present in the commit history.
* **What I learned:** `git log -p` shows the diff of every commit. Even though the password in the current `README.md` was masked as `xxxxxxxxxx`, it had been committed in the clear in an earlier commit — history is never automatically scrubbed.

```bash
git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo
cd repo
git log
git log -p
```

### Level 29 ➔ Level 30

* **Topic:** Reaching information that doesn't appear on the default branch but exists on another one.
* **What I learned:** `git branch -a` lists all branches, including remote ones. The password only existed on the `dev` branch; switching to it with `git checkout dev` revealed the file.

```bash
git branch -a
git checkout dev
cat README.md
```

### Level 30 ➔ Level 31

* **Topic:** Accessing data hidden behind a Git tag.
* **What I learned:** `git tag` lists all tags in the repository; `git show <tag-name>` displays the content of the commit/object a tag points to (here, the password directly).

```bash
git tag
git show secret
```

### Level 31 ➔ Level 32

* **Topic:** Force-adding a file blocked by `.gitignore` and pushing it to trigger a server-side git hook.
* **What I learned:** `git add -f` ignores `.gitignore` rules and force-stages the given file. Once committed with the expected content and pushed, a pre-receive/post-receive hook on the server validated the change and returned the next password.

```bash
echo 'May I come in?' > key.txt
git add -f key.txt
git commit -m "add key.txt"
git push origin master
```

### Level 32 ➔ Level 33

* **Topic:** Escaping a restricted "uppercase shell" that converts all input to uppercase, back to a normal shell.
* **What I learned:** The shell converted every command to uppercase before running it, which made ordinary commands unusable. However, `$0` (a special variable referring to the running shell itself) could be invoked without being uppercased, so `>> $0` launched a fresh, unrestricted `/bin/sh` and bypassed the restriction.

```bash
>> $0
cat /etc/bandit_pass/bandit33
```

### Level 33 (Final)

* **Topic:** The final level of this section of the game.
* **Note:** There's no new task at this level; `README.txt` contains the OverTheWire team's congratulatory message and a pointer to their other wargames.

```bash
cat README.txt
```

---

## 🧠 General Takeaways

* **Filename gotchas** (`-`, spaces, hidden files) are usually solved with a `./` prefix, quoting, or the `-a`/`-la` flags.
* **`find`** is one of the most powerful tools for searching an entire filesystem by size, permissions, or ownership; suppressing error output with `2>/dev/null` is a useful habit.
* **Encoding/compression layers** (hex, base64, ROT13, gzip/bzip2/tar) are typically diagnosed step by step with `file` and reversed one layer at a time.
* **Network services** are reached with `nc` for plain TCP and `openssl s_client` for SSL/TLS.
* **Cron jobs** can be exploited by studying the logic of the scripts they run (predictable filenames, ownership checks, temp-file leaks, etc.).
* **SUID binaries** are a common privilege-escalation vector, granting access to files the current user couldn't otherwise reach, via the binary owner's permissions.
* **Restricted shells** (rbash, an auto-exiting `.bashrc`, uppercase-only input, etc.) usually leave a side door open: an editor's shell-escape feature, a special variable like `$0`, or passing a command directly as an argument.
* **Git** retains not just the latest file contents but the full commit history (`git log -p`), all branches (`git branch -a`), and all tags (`git tag`/`git show`) — once sensitive data has been committed, it remains reachable unless actually purged from history.

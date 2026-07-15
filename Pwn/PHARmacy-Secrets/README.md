# PHARmacy Secrets

## Challenge Information

| Property | Details |
|----------|---------|
| Platform | Pwn.Careers |
| Category | Web Exploitation |
| Vulnerability | PHP PHAR Deserialization (Remote Code Execution) |
| Outcome | Successfully completed |

## Skills Demonstrated

- PHP Deserialization
- PHAR Archive Manipulation
- Remote Code Execution (RCE)
- Web Application Testing
- File Upload Security
- Linux
- HTTP Analysis
- Payload Development
- Problem Solving

## What Was This Challenge About?

This challenge explored a PHP PHAR deserialization vulnerability. PHP PHAR archives can contain serialized objects within their metadata. If an application processes user-controlled PHAR files using the `phar://` stream wrapper, those objects may be automatically deserialized, potentially resulting in Remote Code Execution (RCE).

## Methodology

### Step 1 – Reconnaissance

I first explored the web application to understand how it handled file uploads.

Using `curl`, I inspected the application and confirmed that it accepted PHAR file uploads.

```bash
curl https://ctf.pwn.careers/pharmacy_secrets/
```

The application presented a file upload form for PHAR files.

---

### Step 2 – Understanding the Vulnerability

The challenge hinted that the exploit resided within the PHAR metadata.

I researched how PHP automatically deserializes metadata when a PHAR archive is processed through the `phar://` stream wrapper.

This indicated that successful exploitation would require creating a malicious PHAR archive containing a serialized PHP object.

---

### Step 3 – Building the Malicious PHAR

I created a custom PHP script that generated a PHAR archive containing an object in its metadata.

Several upload attempts were performed using different filenames, extensions and upload techniques to understand how the server validated uploaded files.

During testing I experimented with:

- Different PHAR filenames
- Alternative upload extensions
- Different multipart upload formats
- Custom PHAR stubs

Each failed attempt provided additional information about how the server processed uploaded files.

---

### Step 4 – Confirming PHAR Deserialization

After further testing, I successfully confirmed that the server was processing PHAR metadata.

The application returned an `__PHP_Incomplete_Class` object, confirming that deserialization was occurring but that the required PHP class was not yet defined on the server.

## Exploitation

### Step 5 – Develop the Final PHAR Payload

After confirming that PHAR metadata was being deserialized, I modified the exploit so that the required PHP class was defined within the PHAR stub before the metadata was processed.

This ensured that the serialized object could be successfully reconstructed by the server.

The PHAR archive was then rebuilt using PHP.

---

### Step 6 – Upload the Exploit

The completed PHAR archive was uploaded to the vulnerable application using an HTTP POST request.

The upload was accepted and the server processed the archive.

---

### Step 7 – Execute the Payload

When the application processed the uploaded PHAR file, the PHP object was deserialized successfully.

The payload executed an operating system command that read the flag file from the server.

**Outcome**

The vulnerability was successfully exploited and the challenge objective was achieved.

## Security Recommendations

To reduce the risk of PHAR deserialization vulnerabilities:

- Disable unnecessary use of the `phar://` stream wrapper.
- Never deserialize user-controlled objects.
- Validate uploaded file types using both content and extension.
- Store uploaded files outside the web root whenever possible.
- Keep PHP and third-party libraries updated.
- Avoid exposing detailed error messages that may reveal application internals.

## MITRE ATT&CK Mapping

| Technique | ID | Reason |
|-----------|----|--------|
| Exploit Public-Facing Application | T1190 | The challenge involved exploiting a vulnerable web application. |
| Command and Scripting Interpreter | T1059 | The exploit executed operating system commands through PHP. |
| Exploitation for Client Execution | T1203 | A crafted PHAR archive triggered code execution when processed by the application. |

## Key Takeaways

- PHAR metadata can contain serialized PHP objects that are automatically deserialized when processed through the `phar://` stream wrapper.
- Repeated testing and careful interpretation of server error messages were essential to understanding the application's behaviour.
- Successful exploitation required understanding the order in which PHP executes the PHAR stub and deserializes metadata.
- During a real penetration test, I would review file upload functionality, inspect any use of the `phar://` wrapper, and test whether uploaded archives can trigger unsafe deserialization.

## Lessons Learned

This challenge strengthened my understanding of PHP deserialization vulnerabilities and secure file handling.

Key lessons included:

- PHAR archives can contain serialized PHP objects within their metadata.
- The `phar://` stream wrapper can automatically trigger deserialization when processing uploaded files.
- Error messages can provide valuable clues during vulnerability research but should never be relied upon in production applications.
- Persistence and systematic testing were essential to identifying the correct exploitation path.
- File upload functionality should always be considered a high-risk attack surface during penetration testing.

-  

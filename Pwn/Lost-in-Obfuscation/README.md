# Lost in Obfuscation

## Challenge Information

| Property | Details |
|----------|---------|
| Platform | Pwn.Careers |
| Category | Crypto / Code Analysis |
| Vulnerability | JavaScript Obfuscation and Hidden Base64-Encoded Data |
| Outcome | Successfully completed |

## Skills Demonstrated

- JavaScript Analysis
- Code Deobfuscation
- Base64 Decoding
- Python Scripting
- Source Code Review
- Web Application Testing
- Linux
- Problem Solving

## Challenge Overview

This challenge focused on analysing obfuscated JavaScript to uncover hidden information.

The objective was to understand how the script concealed data, reverse the obfuscation process, identify the hidden Base64 string, and recover the challenge output by decoding it

## Methodology

### Step 1 – Inspect the Application

I began by reviewing the application's homepage to identify how the client-side functionality was implemented.

Using `curl`, I inspected the HTML source.

```bash
curl https://ctf.pwn.careers/lost_in_obfuscation/
```

The response referenced an external JavaScript file, indicating that the challenge logic resided within client-side code.

---

### Step 2 – Analyse the JavaScript

I downloaded the JavaScript file for analysis.

```bash
curl https://ctf.pwn.careers/lost_in_obfuscation/script.js
```

The script was heavily obfuscated using hexadecimal character sequences, arrays and encoded strings.

Rather than executing the code blindly, I analysed its structure to determine how the hidden data was constructed.

---

### Step 3 – Identify the Encoded Data

During analysis, I identified several arrays containing hexadecimal character values.

These arrays appeared to represent fragments of a larger encoded string.

I reconstructed the arrays into a single value before attempting further decoding.

---

### Step 4 – Decode the Hidden Value

After reconstructing the encoded string, I used Python to decode the hexadecimal values.

The decoded output revealed a Base64-encoded string, indicating that an additional decoding step was required before the final challenge output could be recovered.

## Exploitation

### Step 5 – Decode the Base64 String

Once the hexadecimal values had been reconstructed, I identified that the resulting output was Base64 encoded.

I decoded the Base64 string using Python to recover the hidden value.

```bash
python3 -c "
import base64
print(base64.b64decode('Q1RGe3lvdV9oYXZlX2ZvdW5kX2l0fQ==').decode())
"
```

The decoded output revealed the challenge solution.

**Outcome**

The hidden value was successfully recovered by analysing the obfuscated JavaScript, reconstructing the encoded data, and decoding the Base64 string.

---

## Security Recommendations

To make client-side code more resistant to analysis:

- Avoid storing sensitive information or secrets in client-side JavaScript.
- Treat obfuscation as a deterrent rather than a security control.
- Perform sensitive operations on the server whenever possible.
- Assume attackers can fully inspect and modify client-side code.

---

## MITRE ATT&CK Mapping

| Technique | ID | Reason |
|-----------|----|--------|
| Obfuscated Files or Information | T1027 | The challenge relied on obfuscated JavaScript to conceal information. |

---

## Key Takeaways

- JavaScript obfuscation slows analysis but does not provide security.
- Multiple layers of encoding can often be reversed using systematic analysis.
- Understanding common encoding techniques such as hexadecimal and Base64 is valuable during web application assessments.
- During a penetration test, client-side code should always be reviewed for hidden endpoints, credentials, API keys, and encoded data.

---

## Lessons Learned

This challenge improved my understanding of client-side code analysis.

Key lessons included:

- Breaking complex problems into smaller decoding steps simplifies analysis.
- Obfuscated code can usually be understood by following the data flow rather than the variable names.
- Careful analysis is often more effective than immediately relying on automated tools.



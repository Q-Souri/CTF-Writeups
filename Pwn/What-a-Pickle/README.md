# What a Pickle

## Challenge Information

| Property | Details |
|----------|---------|
| Platform | Pwn.Careers |
| Category | Web Exploitation |
| Vulnerability | Python Pickle Deserialization (Remote Code Execution) |
| Flag | CTF{pickle_powered_exploit} |

## What Was This Challenge About?

Python's `pickle` module is used to serialize (save) and deserialize (load) Python objects.

When Python deserializes an untrusted pickle object, arbitrary code can be executed.

This challenge exploited a server that accepted a user-supplied pickle object and deserialized it without any safety checks, resulting in a Remote Code Execution (RCE) vulnerability.

## Skills Demonstrated

- Web Application Security
- Python Deserialization
- Remote Code Execution (RCE)
- Reconnaissance
- Enumeration
- Linux
- HTTP Analysis
- Payload Development
- Problem Solving

## Tools Used

- ping
- nmap
- curl
- gobuster
- Python 3

- ## Methodology

### Step 1 – Reconnaissance

The first objective was to identify the target and understand the application before attempting exploitation.

#### 1. Find the Target IP

I began by resolving the target's IP address.

**Result:** The target was hosted on AWS with the IP address **18.133.228.100**.

#### 2. Scan Open Ports

Next, I performed a port scan to identify exposed services.

**Findings**

| Port | Service | Purpose |
|------|----------|---------|
| 22 | SSH | Remote administration |
| 80 | HTTP | Web server |
| 443 | HTTPS | Secure web server |

#### 3. Explore the Web Application

I used `curl` to inspect the application's homepage and understand the challenge.

```bash
curl https://ctf.pwn.careers/pickled_peril/
```

The response contained clues indicating that the application expected a **Base64-encoded pickle object**.

#### 4. Discover Hidden Paths

I checked `robots.txt` and discovered the hidden **/ctf** path.

From there I:

- Created a CTF account
- Read the challenge description
- Learned the expected payload format
- Identified the initial flag location

#### 5. Locate the Deserialization Endpoint

I tested POST requests until I found the endpoint responsible for deserializing user input.

```bash
curl -X POST https://ctf.pwn.careers/pickled_peril/deserialize \
-d 'data=test'
```

The response confirmed that this endpoint performed deserialization and was therefore the target for exploitation.

## Exploitation

### Step 6 – Create the Malicious Pickle Payload

After identifying the vulnerable endpoint, I created a malicious Python pickle object. The payload executes an operating system command when the server deserializes it.

The payload was encoded in Base64 before being sent to the application.

### Step 7 – Determine the Correct Delivery Method

Through testing, I discovered that the server expected the payload to be sent as a raw `text/plain` HTTP POST body rather than as a form field or JSON payload.

This allowed the application to successfully deserialize the object.

### Step 8 – Locate the Flag

The payload executed the following command to locate the flag file:

```bash
find / -name flag* 2>/dev/null
```

The command identified the flag file on the server.

### Step 9 – Read the Flag

Once the flag location had been identified, I modified the payload to read its contents.

The challenge was successfully completed.

**Flag**

with:

```markdown
**Outcome**

The vulnerability was successfully exploited and the challenge objective was achieved.
```

## Lessons Learned

This challenge reinforced several important penetration testing concepts:

- Never deserialize untrusted data using Python's `pickle` module.
- Error messages can reveal valuable information during reconnaissance.
- Understanding HTTP requests and content types is essential when interacting with web applications.
- Methodical testing and persistence are often more important than knowing the solution immediately.
- Remote Code Execution (RCE) vulnerabilities can have severe consequences when user-controlled data is deserialized.

## Security Recommendations

To mitigate this vulnerability:

- Never deserialize untrusted data using Python's `pickle` module.
- Use safer serialization formats such as JSON whenever possible.
- Validate and sanitize all user-supplied input.
- Apply the principle of least privilege to services handling uploaded or external data.
- Avoid exposing detailed error messages that could assist attackers during reconnaissance.

## MITRE ATT&CK Mapping

| Technique | ID | Reason |
|----------|----|--------|
| Exploit Public-Facing Application | T1190 | The challenge involved exploiting a vulnerable web application. |
| Command and Scripting Interpreter | T1059 | The malicious pickle payload executed operating system commands on the server. |

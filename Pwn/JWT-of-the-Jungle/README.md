# JWT of the Jungle

## Challenge Information

| Property | Details |
|----------|---------|
| Platform | Pwn.Careers |
| Category | Web Exploitation |
| Vulnerability | Weak JWT Secret and Privilege Escalation via Token Forgery |
| Outcome | Successfully completed |

## Skills Demonstrated

- JSON Web Token (JWT) Analysis
- Authentication Testing
- Web Application Security
- Directory Enumeration
- Password Cracking
- Python Scripting
- HTTP Request Analysis
- Linux
- Problem Solving

## Challenge Overview

This challenge focused on assessing the security of a JSON Web Token (JWT) authentication mechanism.

The objective was to analyse how the application handled authentication, identify weaknesses in the token implementation, recover the signing secret, and forge a valid administrator token to gain access to a restricted endpoint.

## Methodology

### Step 1 – Reconnaissance

I began by reviewing the challenge page to understand how the application handled authentication and identify potential attack surfaces.

Using `curl`, I inspected the application's initial response.

```bash
curl https://ctf.pwn.careers/jwt_of_the_jungle/
```

The application indicated that authentication was required before accessing protected functionality.

---

### Step 2 – Obtain and Analyse the JWT

Following the challenge instructions, I authenticated using the provided guest credentials to obtain a valid JSON Web Token (JWT).

Rather than treating the token as opaque, I decoded its header and payload to understand how authorisation decisions were being made.

This analysis identified several important claims, including:

- Username
- Role
- Signing algorithm
- Issued-at timestamp

The presence of a **role** claim suggested that privilege levels were determined directly from the JWT payload, making the token an important target for further testing.

## Exploitation

### Step 3 – Discover Hidden Functionality

Before attempting privilege escalation, I wanted to determine whether any administrative functionality existed that was not exposed through the application's user interface.

I performed directory enumeration using Gobuster.

```bash
gobuster dir -u https://ctf.pwn.careers/jwt_of_the_jungle/ \
-w /usr/share/wordlists/dirb/common.txt
```

This identified an `/admin` endpoint that returned **HTTP 403 Forbidden**, confirming that administrative functionality existed but required additional privileges.

---

### Step 4 – Recover the JWT Signing Secret

Since the token used the **HS256** signing algorithm, its integrity depended on a shared secret.

I attempted to recover this secret using Hashcat with mode **16500**, which is designed for JWT-HMAC tokens.

```bash
echo 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...' > jwt.txt

hashcat -a 0 -m 16500 jwt.txt \
/usr/share/wordlists/rockyou.txt --force
```

The signing secret was successfully recovered, allowing valid tokens to be generated.

---

### Step 5 – Forge an Administrator Token

After recovering the signing secret, I modified the JWT payload by changing the user's role from **user** to **admin**.

The modified payload was then re-signed using the recovered secret, producing a valid administrator token.

---

### Step 6 – Access the Administrative Endpoint

The forged JWT was supplied within the HTTP **Authorization** header.

During testing I observed that the application accepted the token **without** the standard `Bearer` prefix.

Using the forged administrator token, I successfully accessed the restricted administrative endpoint.

**Outcome**

The authentication mechanism was bypassed by exploiting a weak signing secret and forging a valid administrator token with elevated privileges.

## Security Recommendations

To strengthen the security of JWT-based authentication:

- Use a strong, randomly generated signing secret of at least 256 bits.
- Consider asymmetric signing algorithms such as RS256 for improved key management.
- Validate all JWT claims, including issuer, audience, expiration, and issued-at values.
- Never rely solely on client-controlled claims such as `role` for authorisation decisions.
- Monitor authentication logs for repeated token validation failures and brute-force attempts against JWT secrets.

---

## MITRE ATT&CK Mapping

| Technique | ID | Reason |
|-----------|----|--------|
| Exploit Public-Facing Application | T1190 | The assessment targeted a vulnerable web application. |
| Valid Accounts | T1078 | Authentication began using legitimate guest credentials before privilege escalation. |
| Command and Scripting Interpreter | T1059 | Python scripts were used to analyse and forge JWTs during testing. |

---

## Key Takeaways

- JWT payloads are encoded, not encrypted, and should never contain sensitive information.
- Weak signing secrets can allow attackers to forge valid tokens with elevated privileges.
- Hidden endpoints should always be enumerated during web application testing.
- Authentication mechanisms should be assessed alongside authorisation logic to identify privilege escalation opportunities.
- During a real penetration test, I would evaluate JWT implementations for weak secrets, insecure algorithms, missing validation, and trust in client-controlled claims.

---

## Lessons Learned

This challenge strengthened my understanding of authentication testing and JWT security.

Key lessons included:

- JWT security depends on both strong cryptography and secure implementation.
- Enumeration and token analysis are valuable techniques for identifying authentication weaknesses.
- Small implementation flaws, such as weak secrets or excessive trust in token claims, can lead to complete privilege escalation.
- A structured methodology—reconnaissance, analysis, enumeration, exploitation, and validation—helps ensure vulnerabilities are identified systematically.

- 

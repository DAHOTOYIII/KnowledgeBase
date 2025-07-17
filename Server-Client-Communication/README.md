## 1. HTTP (HyperText Transfer Protocol)

**Purpose:**  
Plain‑text, application‑layer protocol for fetching resources (HTML pages, images, JSON, etc.) from a server.

**How it works:**

- **Request:**  
  Client opens a TCP connection (usually on port 80) and sends an ASCII‑based request message:  
  ```http
  GET /index.html HTTP/1.1
  Host: example.com
  User-Agent: curl/7.85.0
  Accept: */*
  ```

- **Response:**  
  Server replies with a status line, headers, blank line, and body:  
  ```http
  HTTP/1.1 200 OK
  Content-Type: text/html
  Content-Length: 1256

  <html>…</html>
  ```

- **Methods:**  
  `GET` (read), `POST` (submit), `PUT` (replace), `DELETE`, `HEAD`, `OPTIONS`, etc.

- **Stateless:**  
  Each request is independent—any session‑state must be managed (e.g., via cookies or tokens).

---

## 2. HTTPS (HTTP over TLS)

**Purpose:**  
Encrypt HTTP traffic to provide confidentiality, integrity, and server authentication.

- **Port:** Default TCP port 443.  
- **Key idea:** Before any HTTP bytes flow, the client and server perform a TLS handshake to agree on encryption keys. Once the secure channel is established, HTTP messages are sent “inside” that encrypted tunnel.

---

## 3. TLS (Transport Layer Security)

TLS sits between TCP and the application (HTTP) layers. It has two main phases:

### 3.1 TLS Handshake

1. **ClientHello**  
   - **Client → Server**  
   - Includes supported TLS version, list of supported cipher suites (e.g. `TLS_AES_128_GCM_SHA256`), and a random nonce.

2. **ServerHello & Certificate**  
   - **Server → Client**  
   - Picks TLS version & cipher suite, sends its X.509 certificate (and any intermediate certs), plus its random nonce.

3. **Certificate Verification**  
   Client checks:  
   - Certificate is signed by a trusted CA.  
   - Certificate’s Common Name (CN) or Subject Alternative Name (SAN) matches the server’s hostname.  
   - Certificate validity dates.  
   - No revocation (CRL/OCSP).

4. **Key Exchange**  
   Depending on the cipher suite:  
   - **RSA key exchange:** client encrypts a “pre‑master secret” with the server’s public key.  
   - **ECDHE (Elliptic‑Curve Diffie–Hellman Ephemeral):** client and server exchange public ECDH points, derive shared secret.

5. **Finished Messages**  
   Both sides derive symmetric keys from the master secret and the nonces, then exchange “Finished” messages authenticated with the negotiated keys to confirm handshake integrity.

### 3.2 TLS Record Layer

- Handles fragmentation, (optional) compression, MAC (message authentication code), and encryption.  
- Once established, application data (e.g. HTTP) is sent as encrypted TLS records.

---

## 4. Certificate (X.509)

An X.509 certificate is a digitally signed statement binding a public key to an identity.

- **Subject:** e.g. `CN=www.example.com, O=Example Ltd, C=US`  
- **Issuer:** the CA that signed it (could be an intermediate CA).  
- **Public Key:** server’s public key for asymmetric crypto.  
- **Validity period:** `Not Before` / `Not After` dates.  
- **Extensions:** key usage, SANs (additional hostnames/IPs), CRL/OCSP URLs, etc.  
- **Signature:** CA’s digital signature over the certificate’s contents.

---

## 5. Certificate Authority (CA)

**Role:** Trusted third party that issues and signs certificates.

**Trust chain:**

1. **Root CA:** Self‑signed, distributed in client trust stores (browsers, OS).  
2. **Intermediate CA(s):** Signed by the root, issue end‑entity (server) certificates.  
3. **End‑entity certificate:** Signed by an intermediate CA.

**Verification:**  
Client builds a chain from the server’s certificate back to a trusted root and cryptographically verifies each signature.

---

## 6. End-to-End Client–Server Sequence

When you type `https://www.example.com/` in your browser, the steps are:

1. **DNS Resolution**  
   Browser asks your DNS resolver: “What’s the IP of `www.example.com`?”  
   Resolver → returns e.g. `93.184.216.34`.

2. **TCP 3‑Way Handshake**  
   - Client → Server (SYN)  
   - Server → Client (SYN‑ACK)  
   - Client → Server (ACK)

3. **TLS Handshake** (see section 3.1)

4. **HTTP Request inside TLS**  
   ```http
   GET / HTTP/1.1
   Host: www.example.com
   ```
   All bytes are encrypted.

5. **HTTP Response inside TLS**  
   ```http
   HTTP/1.1 200 OK
   Content-Type: text/html

   <html>…</html>
   ```

6. **Connection Close**  
   Either side sends a TCP `FIN/ACK` or uses a TLS `close_notify` alert before tearing down the connection.

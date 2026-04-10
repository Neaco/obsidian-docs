At the client device boundary, the local TLS trust store is responsible for **deciding whether the remote identity is trusted**

1.  **Root of trust**
- Stores trusted **root and intermediate CAs**
- Defines *who is allowed to sign certificates the client will accept*

2. **Certificate validation**
During the TLS handshake, the client : 
- Verifies the certificate chain up to a trusted CA
- Checks the certificate validity period
- Verifies hostnames matches (CN / SAN)
- Rejects revoked or untrusted issuers (when revk enforced)

3. **Policy enforcement** 
- Enforces local security policy : 
  - allowed TLS versions
  - allowed cipher suites
- Corporate may inject their own root CA to intercept TLS
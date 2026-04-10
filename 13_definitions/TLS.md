#TLS

Sits between L4 and L7 ([[Network layers]]) and verifies the identity of the other party before starting L7 communication.

It provides:
- **Encryption** (confidentiality)
- **Authentication** (certificates)
- **Integrity** (no tampering)



### TLS Handshake

Simplified flow :
1. Client opens a TCP connection (L4)
2. TLS handshake starts:
   - Client says: "I want HTTPS from example.com"
   - Server sends certificate
   - Client verifies trust chain
3. Secure channel is established 
4. **only then** HTTP flows inside TLS




#### TLS Termination

termination = where encryption ends

Common patterns:
**1) TLS terminated at the edge**
- Cloudflare decrypts
- Talks plaintext or re-encrypted TLS to origin

Pros:
- centralized certs
- WAF visibility

Cons:
- You trust the edge completely


**2) TLS terminated at ingress**
- CLoudflare passes TLS through
- Ingress holds certs

Pros:
- Full end-to-end encryption
- App-aware routing

Cons:
- More cert management

**3)TLS end-to-end (rare)**
- App handles TLS itself

Pros: Strong isolation

Cons: Operationnally heavy




![[Local TLS trust store]]


![[TLS interception]]
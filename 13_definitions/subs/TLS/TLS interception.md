Steps : 
0. Inject your own root CA to the client device's **trusted root store**
When client connects to https://example.com:
1. Intercept request to example.com 
2. Intercept TLS connection
3. Generate your own certificate for example.com with your own root CA (added in step 0)
4. Decrypt the request
5. Read the request
6. Re-encrypt it and send it to example.com
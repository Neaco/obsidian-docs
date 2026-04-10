# L4 vs L7

- L4 Load Balancing is fast and simple : messages are neither inspected nor decrypted. However, it cannot route traffic based on content.
- L7 Load Balancing makes decisions based on the actual content of each message. Can decrypt and inspect messages and make content-based routing decisions, initiate TCP connection to the appropriate upstream server.
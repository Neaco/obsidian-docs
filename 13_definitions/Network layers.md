<img src="https://cdn2.hubspot.net/hubfs/2954816/The%207%20Layers%20of%20OSI.png">

## Layer 3 - Network (IP)

Question it answers : 
	"Where should this packet go ?"

- Uses IP addresses
- Responsible for routing between networks
- Does not care if the destination is alive or listening

Ex: VNet routing

## Layer 4 - Transport ([[TCP]]/UPD)

Question it answers : 
	"Is there a process listening, and can we exchange bytes reliably?"

- Uses ports
- TCP handles : 
  - Connection setup (SYN / ACK)
  - retransmission
  - ordering
- Load balancers at L4 do not understand HTTP

Ex: Health probes

## Layer 7 - Application (HTTP)
Question it answers : 
	"What does this request mean ?"

- Understands:
  - URLs
  - headers
  - status codes
- Can make routing decisions based on content

Ex: API gateway



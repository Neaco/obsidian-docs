# Context

- The domain name has been purchased on OVH. 
- We use Cloudflare as edge of Internet
- We will need to make frequent changes to the DNS config

# Decision
- Migrate **manually** the domain to Cloudflare's DNS
- Update the DNS config through terraform


# Options considered
1)  Update DNS manually
2) Keep the DNS on OVH and use the OVH terraform provider


# Decision drivers
- Programmatically manage the DNS
- Use the more widespread solution for experience


# Consequences
## Positive
- One less partner and terraform dependency
- Use of a more complete solution

## Negative
- Manual action required


# Migration steps
- Create Cloudflare zone
- Retrieve the DNS servers for that zone
- In OVH, modify the DNS servers and set the ones provided by Cloudflare
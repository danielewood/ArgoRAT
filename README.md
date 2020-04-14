# ArgoRAT
Argo Tunnel Remote Access Tool

## Concept
- Work-in-Porgress

### Client (golang wrapper for cloudflared or just import the sources of cloudflared and add module)
- Gets config from server via HTTPS (token or some other UUID for config mapping on server database)
- Config:
  - yaml/JSON of (many) port:service:proto:targethost:comment
  - client FriendlyName
  - client UUID
- Spawns multiple `cloudflared tunnel` with config from above
- Puts status json to server with current list of tunnels and urls.

### Server
Docker/Linux VM
- Database to store client configs and connection states
- Bridged/Internally NATted NIC to bind arbitrary IP addresses for port forwarding
- DNS with suffix of .argo.domain.com
  - ${FriendlyName}.name.argo.domain.com
  - ${UUID}.uuid.argo.domain.com
- nginx to rewrite all incoming http/https bound requests to the target argo url

### Desired outcome

ClientConfig.yml:
```yaml
FriendlyName: ClientName
UUID: 7e03c4fe-2b21-42da-af88-77954ce27d98
tunnel: 1
  - clientport: 23
  - relayport: 23
  - targethost: localhost (or 127.0.0.1)
  - service: telnet
  - proto: tcp
  - comment: documentation

tunnel: 2
  - clientport: 23
  - relayport: 10023
  - targethost: host_reachable_from_client.domain.com
  - service: telnet
  - proto: tcp
  - comment: documentation

tunnel: 3
  - clientport: 80
  - relayport: null
  - targethost: host_reachable_from_client.domain.com
  - service: webproxy
  - proto: http
  - comment: configures server nginx proxy_pass to do header rewrites of ${UUID}.uuid.argo.domain.com to aaaa-bbbb-cccc-dddd.trycloudflare.com
  
tunnel: 4
  - clientport: 445
  - relayport: 445
  - targethost: host_reachable_from_client.domain.com
  - service: smb
  - proto: tcp
  - comment: allows client on lan local to server to connect to smb://${UUID}.uuid.argo.domain.com/share

tunnel: 5
  - clientport: 1234
  - relayport: 1234
  - relaytarget: bbbb-cccc-dddd-eeee.trycloudflare.com
  - service: netcat
  - reverse: true
  - proto: tcp
  - comment: causes server to create its own reverse tunnel and share details with the client
```

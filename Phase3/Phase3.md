# Phase3 : The Networking

### Traefik, Pi-hole, Unbound, DNS & Tailscale

## 1. Moving to a Proper Network Setup

After getting the Docker services and monitoring stack running, I wanted to improve how I accessed and managed everything on the homelab. Opening each application using its IP address and port was becoming reaallly boring, and I wanted to use proper domain names instead.

The plan was to build a network setup around three main components:

- **Traefik** for reverse proxying and HTTPS.
- **Pi-hole** for local DNS and domain blocking.
- **Unbound** for recursive DNS resolution.

I also wanted to keep remote access available through Tailscale, without making every service publicly accessible.


## 2. Getting Traefik Working

I chose Traefik instead of Nginx because everybody on reddit was saaying nginx proxy manager is a hot smoking garbage, jk, i wanted a reverse proxy that have the complexity to have control over what you wanted plus it works really well with docker containers.

The idea was to create a central entry point for the services and stop exposing individual application ports wherever possible.

### The Docker API error

During setup, Traefik reported an error similar to:

```text
client version 1.24 is too old.
Minimum supported API version is 1.40
```

This was a useful lesson in Docker compatibility. Even when a container starts, it still needs a compatible API to communicate with the Docker daemon. The problem was not necessarily the routing configuration itself.

I had to investigate the compatibility between Traefik and the Docker environment rather than immediately changing the routers or labels.

### Creating the proxy network

I used a Docker network named `proxy` to connect Traefik to the applications it needed to reach.

Later, Docker Compose produced a warning that the network already existed but had not been created by the current Compose project. Traefik also reported that it could not find the expected network for Grafana.

This was confusing because inspecting the Traefik container showed a network actually named `proxy`.

The lesson was that Docker Compose project ownership and the actual Docker network are two separate things. An existing network can be perfectly valid, but Compose needs to be told whether it should manage that network or use it as an external resource.

### HTTPS and Let`s Encrypt

The next challenge was obtaining a trusted HTTPS certificate for my domain.

I used Traefik's built-in ACME integration. 

### The relationship between the components is:
```
Let's Encrypt: the certificate authority that issues publicly trusted certificates.

ACME: the protocol used to request, validate, issue, and renew certificates.

Traefik: the ACME client that manages certificates for my services.

deSEC: the DNS provider used to manage the DNS records and support DNS-based domain validation.
```
---
```

Why I chose DNS-01 validation

There are several ways to prove control over a domain. HTTP-01 validation requires the certificate authority to reach a specific HTTP endpoint. DNS-01 validation instead requires a particular TXT record to appear in DNS.

I configured Traefik to use the DNS-01 challenge through deSEC.

This was especially useful because I wanted wildcard coverage for my homelab subdomains. A wildcard certificate can cover names such as grafana.gogoli.dedyn.io and proxmox.gogoli.dedyn.io, provided the certificate includes *.gogoli.dedyn.io.

The DNS-01 process works approximately like this:

Traefik
   |
   | Requests a certificate
   v
Let's Encrypt ACME server
   |
   | Requests proof of domain control
   v
Traefik creates a DNS TXT record through deSEC
   |
   | Let's Encrypt checks the public DNS record
   v
Domain ownership validated
   |
   v
Certificate issued
   |
   v
Traefik serves HTTPS

The TXT record used for validation is temporary validation data, not the same thing as the A record used to direct clients to a service.

Another important distinction: DNS-01 validation does not require the web application to be publicly reachable. Let's Encrypt checks the DNS challenge rather than connecting to the application itself. This makes it suitable for obtaining a publicly trusted certificate for a service that I intend to keep private.
```

Once the reverse proxy configuration was in place, I started routing applications through domain names under `gogoli.dedyn.io`. Because dedyn.io was giving a free domain.

Grafana was configured using Traefik labels, including the `websecure` entry point, TLS, and the container's internal port `3000`.

I also configured a route to the Proxmox web interface through Traefik using the file provider, with the existing access-control middleware.

This introduced another important distinction: a certificate can make a connection encrypted, but it does not decide who is allowed to connect. TLS and access control need to be configured separately.

## 3. Connecting the Tailscale
After configuring the basics i started to look for alternatives to setup remote access, there were several but i have so little control of my router so i just went with tailscale tunnel.

I then succesfully connected everything to eachother with access to ssh.



## 3. Adding Pi-hole

After working on the reverse proxy, I moved on to local DNS. Tailscale`s DNS would have been enough? Yes, but SELF-HOSTING BABYYY!

I wanted to type a service's domain name into a browser and have it reach Traefik directly, without relying on the public DNS answer or manually remembering an IP address.

Pi-hole was set up on a separate Raspberry Pi i had laying around 

The intended setup was for Pi-hole to resolve local service names to the Traefik host. Traefik would then decide which application should receive the connection.

This meant that DNS needed to return the correct **local address**, not just any valid address associated with the domain.

### The unexpected DNS result

While testing the setup, I queried Pi-hole directly from Windows:

```powershell
nslookup grafana.gogoli.dedyn.io 192.168.0.111
```

I expected the result to be `192.168.0.103`, but the response was:

```text
Name:    grafana.gogoli.dedyn.io
Address: 100.100.158.97
```

The returned address was the Tailscale-side address associated with the existing setup, rather than the expected LAN address.

This was the point where the DNS configuration became more complicated than simply adding a record.

### Troubleshooting the DNS issue

I investigated several possible causes:

- Whether Pi-hole was actually receiving the query.
- Whether its local DNS record was correct.
- Whether a cached or forwarded answer was being returned.
- Whether Windows was using different DNS servers on Ethernet and Wi-Fi.
- Whether Tailscale or a Windows DNS policy was influencing name resolution.
- Whether the router or another DNS configuration was interfering.

The Windows machine had both Ethernet and Wi-Fi interfaces,plus i couldnt know if it is coming from ipv4 or ipv6 which made DNS troubleshooting more confusing. Different interfaces can have different DNS settings, and Windows may use more than one network path.

I considered setting both interfaces to use Pi-hole and flushing the Windows DNS cache. However, the direct query to `192.168.0.111` still returned the unexpected address, so changing the client settings alone could not be treated as the confirmed solution.

The next useful step was to check Pi-hole's Query Log while repeating the request. If the request appeared there with the wrong answer, the investigation would focus on Pi-hole's records and forwarding behavior. If the request did not appear, the problem would be somewhere along the path between the client and Pi-hole.

**Status at this point:** the source of the unexpected DNS response still needed to be confirmed. I did not consider the LAN DNS setup complete.

After a while it started working persistently so it was the correct setup but i still dont know what caused the wrong ip.

### What I learned

The most important lesson from this problem was that a successful DNS response does not necessarily mean the correct DNS server or record was used.

I also learned to test the DNS server directly before changing several settings at once. Otherwise, it becomes difficult to determine which change actually affected the result.

## 4. Adding Unbound

I also configured Unbound as the recursive resolver behind Pi-hole.

The intended local endpoint was:

```text
127.0.0.1:5336
```

The idea was to keep Pi-hole responsible for filtering and local DNS records while Unbound handled recursive resolution for external domains.

This setup taught me to think about DNS as a chain of services rather than a single setting. A client can reach Pi-hole successfully, but external resolution can still fail if Pi-hole cannot communicate with Unbound. Likewise, Unbound can work correctly while Pi-hole's local records remain wrong.

For that reason, each part needs to be tested independently:

1. Can the client reach Pi-hole?
2. Does Pi-hole return the correct local records?
3. Can Pi-hole forward external queries to Unbound?
4. Can Unbound resolve external domains?

I kept Unbound bound to loopback because it was intended to serve Pi-hole locally, not to become another DNS server exposed to the entire network.

## 5. Splitting DNS

One of the biggest complications in the network setup was figuring out how the same domain names should behave on the local network and when connecting remotely through Tailscale.

Initially, `gogoli.dedyn.io` resolved to `100.100.158.97`, the Tailscale-side address associated with the setup. However, when accessing services from home, I wanted the domain names to resolve directly to the local Traefik host at `192.168.0.103`.

The idea was to use **split DNS**: keep the same service hostname, but return a different IP address depending on where the DNS query is being resolved.

### The intended setup

- **On the LAN:** Pi-hole resolves local service names, such as `grafana.gogoli.dedyn.io`, to the local Traefik address `192.168.0.103`.
- **Remotely through Tailscale:** The existing Tailscale DNS configuration continues to provide the address appropriate for remote access.

This would let me use the same domain names in both situations without manually switching between local IP addresses and remote addresses.

### The problem

**The problem is described in the 3rd section**


## 6. Current State and Next Steps

At this stage, the main network components and their intended responsibilities are established:

- Traefik is the central reverse proxy for the web applications.
- The `proxy` Docker network connects Traefik to its backend services.
- Pi-hole provides local DNS and filtering.
- Unbound provides recursive DNS resolution behind Pi-hole.
- Tailscale remains part of the remote-access setup.
- deSEC is used in the existing domain and certificate configuration.



The next step is to conduct some test, strengthen the security, implement proper backups and redundancy. 

I am also thinking of removing the proxmox and dedicating the laptop to the ubuntu server and making a separate and proper NAS for all of it with Pi or another laptop idk we`ll see.


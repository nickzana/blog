+++
title = "Backups - Time Machine over Tailscale"
weight = 2
[taxonomies]
tags = ["security", "docker", "tailscale"]
+++

[`tailscale-timemachine`](https://git.nickzana.dev/nick/tailscale-timemachine) uses Tailscale networking to integrate a modern SSO provider with Time Machine for simple and secure remote backups. It is deployable as a set of containers that connect to any Tailscale network.

# Time Machine

Time Machine is an incremental backup program shipped as part of macOS. It offers a simple, built-in method to backup and recover versions of any file backed up as part of the system. The user interface works seamlessly within macOS and requires little to no interaction from the user.

Unfortunately, Samba is the only network file sharing protocol officially supported by Time Machine. Like many legacy protocols, its authentication is an outdated username and password based approach that does not support modern security tools such as SSO or multi-factor authentication. 

Beyond simply being less secure than an SSO provider, a username and password are an additional set of credentials for users to configure and keep track of. It adds friction and complexity to the backup onboarding and data recovery processes. Using a web-based SSO solution that protects all resources is a more familiar and streamlined experience for users, particularly if an organization already has an identity provider.

# The Solution: Tailscale

Tailscale (or its self-hosted alternative, Headscale) integrates with any SSO provider that supports OAuth2, which makes it ideal for integrating with existing infrastructure. Since authentication happens within the web browser, where most of a user's credentials already are, users have access to password managers or modern APIs such as WebAuthn, further increasing the available security measures.

As an overlay network, Tailscale uses Access Control Lists (ACLs) to define which resources in the network a node can access. By creating a new Tailscale node on the backup server for each user who uses Time Machine in the network, authentication can be effectively enforced at the network level, completely bypassing the need for Samba's built-in authentication.

The client for this project needed to back up machines remotely. While incremental backups are relatively small, initial backups and, critically, system recovery benefit from maximizing the bandwith of the connection. Beyond simply providing authentication, Tailscale simplifies routing the Time Machine traffic over the fastest possible path. 

Tailscale uses a technique called NAT-Traversal to create direct connections between individual nodes. Whether the client machine has a direct 10 Gbps connection to the server or is on a weak WiFi network thousands of miles away, Tailscale exposes the Samba share to the client at the same host address, leaving Time Machine to scale with the available bandwidth regardless of the current network conditions.

# Networking challenges

## Inter-container communication

While Tailscale handles routing traffic between nodes on the network, there are still some remaining challenges getting the Samba share connected to it. 

`tailscale-timemachine` uses the reverse-proxy Caddy, along with the `caddy-tailscale` module, to bind to each hostname on the Tailscale network and route traffic to the corresponding Samba shares. In order for traffic to be routed directly from the client nodes to the Samba share, `caddy-tailscale` currently chooses a random port to listen on. Unfortunately, there is no mechanism for specifying a specific port, meaning the correct port to forward to the container cannot be known ahead of time. 

As a consequence of this, the container that binds to the `tailscale` network must be in "host" networking mode so that the chosen port is automatically forwarded to the container. Unfortunately, this causes a problem for us. Due to a limitation of the Docker networking model, containers in the "host" networking mode cannot be a member of Docker bridge networks, which are the typical method of inter-container networking.

In order to get around this, `tailscale-timemachine` requires manually specifying an IP for each docker container so that the Caddy container can forward traffic to it.

In a `docker-compose.yml` file, each Time Machine Samba share container is defined like so:

```yaml
version: "3"
services:
  tm-1:
    image: mbentley/timemachine:smb
    container_name: tm-1
    volumes:
      - $TM_PATH/tm-1:/opt/timemachine
      - ./smb.conf:/etc/samba/smb.conf
    environment:
      - DEBUG_LEVEL=2
      - CUSTOM_SMB_CONF=true
    networks:
      expose:
        ipv4_address: 172.24.0.11
    restart: unless-stopped
```

## Raw TCP Forwarding

Due to its ubiquity, most modern software services use the HTTP protocol to communicate. As a result, it's generally the most supported protocol for tools interacting with reverse proxies like Caddy. This complicates putting a reverse-proxy like Caddy in front of  Samba server, as Samba communicates over a raw TCP socket, not the HTTP protocol.

While Caddy lacks native support for TCP socket forwarding, there is another third-party module called "layer4" that enables TCP port forwarding. This limits `tailscale-timemachine`'s ability to use many tools, including the Authentication module built into `caddy-tailscale`.

In order to add a Time Machine share, a new server is defined to route traffic from the specified host to the Samba container IP address, like so:

### `config.json`
```json
{
  "apps": {
    "layer4": {
      "servers": {
        "tm-1": {
          "listen": ["tailscale/tm-1:445"],
          "routes": [
            {
              "handle": [
                {
                  "handler": "proxy",
                  "upstreams": [
                    {"dial": ["172.24.0.11:445"]}
                  ]
                }
              ]
            }
          ]
        }
      }
    }
  }
}
```


# Future Improvements

## Docker network simplification

The first and simplest thing to solve would be adding a method to specify the port `caddy-tailscale` binds to. If this was implemented, the Caddy container could forward that particular port from the host and would no longer require "host" networking mode. As a result, it could directly join the docker bridge network, eliminating the need to manually specify IPs for each container.

`tailscaled`, the software that powers most Tailscale clients, already has a method for specifying a port, so it should be possible to modify the code of `caddy-tailscale` to support this.

## Tailscale AuthKey Provisioning

Currently, the AuthKeys required to add the Time Machine Samba nodes to the network must be manually provisioned. Integrating with the Tailscale and Headscale APIs to automatically provision the API keys would reduce the manual intervention required from the server adminitrator to add a Time Machine share.

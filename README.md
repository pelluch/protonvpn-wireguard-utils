# protonvpn-wireguard-utils

Keeps a ProtonVPN port forward alive over NAT-PMP and points qBittorrent at it.

`protonvpn-port-forward` renews the mapping every 45 seconds, since Proton drops it after 60. When the forwarded port changes, it sets qBittorrent's listening port through the WebUI API.

## Requirements

- A ProtonVPN connection to a P2P server, with NAT-PMP enabled
- `natpmpc` (libnatpmp), `curl` and `jq`
- qBittorrent 5.2 or newer, with an API key (WebUI → Options → Web UI)

## Setup

Fill in `/etc/protonvpn-wireguard-utils/protonvpn-port-forward.conf`, then:

    systemctl enable --now protonvpn-port-forward

It only logs when a port changes or something fails:

    journalctl -u protonvpn-port-forward
    journalctl -u protonvpn-port-forward -p err

## How Proton's NAT-PMP behaves

Proton doesn't document most of this, so it's from testing:

- The symmetrical request gets a random port, and traffic arrives locally on that same port. TCP and UDP get the same one.
- Each `PORTS_TO_FORWARD` entry gets a random public port forwarded to the local port you give it. You can't choose the public port; it shows up in the log.
- You get 5 of those on top of the symmetrical one. TCP and UDP for the same local port share a slot, so `8080:tcp` and `8080:udp` count once.
- Ports stay reserved for a while after a mapping lapses, so a short outage gets you the same port back. It also means an entry you just removed keeps its slot until it ages out.
- `CLEAR_PORTS=true` wipes every mapping, the symmetrical one included, so every start, stop and restart gets new ports.
- If you test a forwarded port from your own network, connections arrive from `10.2.0.1`. From anywhere else they keep their real address.

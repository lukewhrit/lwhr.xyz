---
title: "My first homelab"
date: 2026-09-01
---

This summer, I bought a used Dell PowerEdge R730 off of Facebook Marketplace for about $300. It had two Xeon E5-2670 v3 chips, 32GB of ECC ram, rails, eight drive caddies, and an LSI card already flashed into IT mode. I drove home about an hour up the Garden State Parkway with a 2U server and realized I really did not know what I was getting myself into.

This was going to be my first year in my new college apartment and my goal was to be able to run and own everything myself. Router and firewall, network attached storage, media server, local AI models, productivity software, and give projects like [Spacebin](https://spaceb.in) a new public home.

This is the honest version of how that project went.

## Why enterprise hardware

When I was younger, I watched a lot of videos about other people's homelabs, and back then, enterprise hardware was what you did. So I think that was already in my head when I made the decision to use this stuff.

Truth be told, a mini PC would've been more energy efficient and a lot quieter. A tower build would've had similar benefits and is also simpler. But I wanted twelve drive bays I could grow into, real remote management through iDRAC, and the kind of hardware that I could learn about real enterprrise networks on. The R730 gave me all of that for less than a new mini PC.

Thankfully, with my current setup and energy being pretty cheap in Worcester, the R730 has only costed about $20 in electricity since July and stays quiet enough.

## The software stack

Proxmox runs on the bare metal, with a mirrored pair of SATA SSDs holding
the OS and VM disks. On top of that:

- **OPNsense** as a VM, doing all the routing, firewalling, and DNS
- **Coolify** managing a pile of Docker services
- **Jellyfin** for media, with the whole Sonarr / Radarr / Prowlarr / qBittorrent stack feeding it
- **Spacebin**, my pastebin, exposed to the public internet through a Cloudflare Tunnel
- Network-wide ad blocking through OPNsense's Unbound resolver

### Future plans

Down the road, I want to upgrade with a GPU and start running local AI models. Once memory prices go down, I'm going to start hosting game servers for my friends and I. And, I need to get around to setting up Nextcloud and immich to replace the last few services I'm paying monthly for.

## The part where nothing worked

If you take only one thing away from this post, let it be that building something like this is 99% debugging. Even though it's the most tedious, it's also the part where you learn the most.

**The config wouldn't stick.** Early on, OPNsense kept losing its network configuration on every reboot. I'd set the LAN IP, it would work, I'd reboot, and it would be gone. I chased permissions, I chased disk space, I chased the DHCP server. The actual cause was embarrassingly simple: the VM was still booting from the installer ISO instead of the disk it was installed to. It had been running in a live environment the whole time, so nothing it saved survived a restart. Remove the ISO from the boot order and suddenly everything persisted. Hours of my life for one dropdown.

**The throughput that cratered.** After the network cutover, my download speed through OPNsense dropped to 0.19 Mbps. Not slow, broken. The modem itself tested at 940 down, so the problem was mine. The fix was a single tunable: disabling LRO on the virtio NICs. On a virtualized router, the default offloading mangles inbound packets badly enough to retransmit almost everything. One line, and it jumped back to normal.

**The wrong access points.** While I was buying the accessory hardware for the network setup, I came across an amazing deal: 3 Wifi 6 access points for $45, a 48-port switch for $15. I guess I knew there must've been a catch but, on the off chance there wasn't, I couldn't pass up a deal like that. Turns out, there most definitely was a catch. The APs were completely locked down. They were taken from restaurants and only could be configured by [Toast POS technicians](https://pos.toasttab.com/). The network switch works well but is essentially unmanaged since it's control panel is just as locked down, but that's fine for my needs right now.

**The outage that wasn't.** My favorite one. The whole house "lost" local network access while the internet still worked. I spent a while convinced OPNsense had half-crashed, routing but not answering. The DNS server my laptop reported belonged to my university. I was connected to my school's VPN. Nothing was broken. I'd been debugging a problem that didn't exist for a solid stretch. Always check whether you're tunneled somewhere before you tear your own network apart.

## What I'd tell past me

1. Take a snapshot the moment something works. I lost rebuild time more than I could count because I didn't have a clean rollback point. Proxmox snapshots would've cost nothing and saved my afternoon.

Keep permissions in mind every time you touch a Docker volume. When I consolidated and realigned my mounts, the ownership didn't carry over, and the containers running as a non-root user silently couldn't write. The same `chown` fixes it every time, but only if you remember it's the cause.

And consolidate your related services into one compose file early. When I was setting up my media stack, and again when I was setting up Spacebin/CF Tunnels, I fought container-to-container networking across separate Coolify deployments for longer than I should have. One stack, one network, service names that resolve. The whole class of problem disappears.

## Where it is now

It runs. The router routes, the firewall filters, ads die at the DNS level across every device in the house, my media library organizes itself, and Spacebin serves the public internet from a 2U box in my rack without exposing anything else on my network. I built it, I broke it a dozen ways, and I understand every layer because I had to fix every layer.

That last part is the whole point. You can rent all of this. I wanted to own it, and more than that, I wanted to know how it works (and I hoped I could save money long-term). Now I do.

Right now, it's sitting in my dining room. It's not ugly but I started working on a plan to hide it. I'm going to custom design a wooden cabinet with false mesh cabinets that allow air to pass through and out through a vent on top, as well as a regular drawer to store stuff like printer paper.

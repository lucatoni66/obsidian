---
title: "Self-Host Your Homelab Tunnels On bunny.net"
source: "https://alec.is/posts/cloudflare-tunnels-on-bunny-net/"
author:
published:
created: 2026-06-18
description: "Principal software engineer with deep experience in full-stack TypeScript, custom web browsers and real-time, media-focused systems."
tags:
  - "clippings bunny"
---
As a long-time Cloudflare user and advocate, it hurts me to admit that I’ve been pretty unsatisfied with them lately. For three reasons:

1. Their unwillingness to increase transparencyFor years, Cloudflare has notably lacked both billing forecasting and prepayment options. As a customer, you’re essentially forced to accept whatever charges appear on your statement after the fact. in their pricing model.
2. Their chaotic product directionThe user experience within the Cloudflare dashboard is, frankly, a mess. The platform suffers from inconsistent naming conventions that make it difficult to distinguish between services, and many products are fragmented across multiple dashboards with overlapping or conflicting settings. and roadmap.
3. Their toxic cultureCloudflare’s reputation for toxic culture is well-known. How they treat internal employees, loyal customers, and the open source community speaks for itself..

So this month, I’ve been migrating a bunch of my projects over to bunny.net—one of Cloudflare’s strongest competitors—and for the most part, it’s been a breeze. It’s like rewriting Cloudflare from the ground up. bunny.net is what you’d get.

![](https://alec.is/posts/cloudflare-tunnels-on-bunny-net/images/meme.png)

NO CLOUDFLARE TUNNELS?!??

- Cloudflare Workers/Pages → bunny.net Scripts ✅
- Cloudflare Containers/Sandbox → bunny.net Magic Containers ✅
- Cloudflare D1 → bunny.net Database ✅
- Cloudflare R2 → bunny.net Storage ✅
- Cloudflare DNS → bunny.net DNS ✅
- Cloudflare Tunnels →??? ❌
- Cloudflare Email →??? ❌

bunny.net, bunny.net, bunny.net! But wait—what about Cloudflare Tunnels?

![NO CLOUDFLARE TUNNELS?!??](https://alec.is/posts/cloudflare-tunnels-on-bunny-net/images/choc.webp)

NO CLOUDFLARE TUNNELS?!??

We’re homelab people. Of course we need tunnels. I’m not going to be opening a port on my firewall for anything other than WireGuard.

So, as a bunny.net migration exercise, I decided to build a self-hosted Cloudflare Tunnels alternative that runs natively on bunny.net. Let’s go.

#### The Plan

- Make it `frp` -based
	- Use [`@fatedier/frp`](https://github.com/fatedier/frp) as the backbone. This gives us the tunnel relay server (`frps`), the client (`frpc`), and web dashboards. If you haven’t seen `frp`, check it out—it’s awesome. tl;dr: it’s an open-source, self-hosted, multi-site reverse proxy similar to Pangolin or `cloudflared`.
- Make it Docker-based
	- Build a multi-arch, server/client-agnostic Docker image that runs the relay in a Magic Container and the client in your homelab (or anywhere).
- Provide good docs
	- Since bunny.net doesn’t have a community-driven template ecosystem like Cloudflare Workers/Pages, good setup docs are non-negotiable.

#### The Result

This went way better than I expected. I’ve released it as [`@alectrocute/tunbun`](https://github.com/alectrocute/tunbun), available publicly on Docker Hub. Check out the code—or keep reading for a quick setup guide.

##### Step 1: Deploy the Magic Container

- Quick Deploy
- Search for `alectrocute/tunbun`

You’ll be prompted to configure the container. Add an additional Anycast IP (`endpoint-3`, `80 -> 80`) and set the minimum environment variables.

`TUNBUN_TOKEN` is a long string used to authenticate client and server—keep it secure.  
`TUNBUN_DASHBOARD_PASSWORD` is the admin password for the dashboard.

![](https://alec.is/posts/cloudflare-tunnels-on-bunny-net/images/dash-config.png)

Create the Magic Container. While it spins up, note the `80 -> 80` Anycast IP—you’ll need it for your per-app pull zone configs.

Once it’s live, you should be able to access the dashboard at:

`http://<your-container-anycast-ip>:7500`

From there, you can see the server’s status, clients, proxies, individual virtual hosts, bandwidth, and more—freakin’ epic.

![](https://alec.is/posts/cloudflare-tunnels-on-bunny-net/images/frp-server.png)

##### Step 2: Configure your per-app pull zones

This part is **very** important.

For each app you want to expose, configure a pull zone. So something like `my-media-player.b-cdn.net`. Point your app’s domain to the `80 -> 80` Anycast IP you noted earlier (**not** the `*.bunny.run` or `*.b-cdn.net` domains), and make sure to enable “Forward host header.”

![](https://alec.is/posts/cloudflare-tunnels-on-bunny-net/images/pull-zone.png)

Optionally, disable caching, you’ll probably want to do this for most of your apps unless it’s a static site or something.

![](https://alec.is/posts/cloudflare-tunnels-on-bunny-net/images/pull-zone-cache.png)

This is also a good time to set rate limits, enable bunny.net Shield, redirect requests to everything that’s not your per-app pull zone(s) and turn on any other neat WAF security features you care about.

##### Step 3: Configure your client(s)

Now in your homelab which assumably is running a bunch of Docker containers on a Linux host, set up the client(s) to connect to the relay, just like you used to do with `cloudflared`:

```yaml
services:
  tunbun-client:
    image: alectrocute/tunbun:latest
    network_mode: host
    environment:
      TUNBUN_MODE: client
      TUNBUN_SERVER_ADDR: 109.224.229.212
      TUNBUN_SERVER_PORT: 7000
      TUNBUN_TOKEN: change-me
      TUNBUN_LOCAL_PORT_TO_FQDN: "4002:my-media-player.b-cdn.net,8080:my-pihole-for-example.b-cdn.net"
      TUNBUN_DASHBOARD_PORT: 7500
      TUNBUN_DASHBOARD_USER: change-me
      TUNBUN_DASHBOARD_PASSWORD: change-me
```

You can check the `frp` client dashboard at `http://localhost:7500`. This’ll allow you to see the client’s status, logs, config and even update the config in real-time (with option to persist to a file). All of the functionality of `cloudflared` and so much more.

![](https://alec.is/posts/cloudflare-tunnels-on-bunny-net/images/dashboard-client.png)

##### Step 4: Test your setup

Now, if you go to `http://my-media-player.b-cdn.net`, you should see your media player, or whatever you chose to expose.

If you’ve used Cloudflare Tunnels before, this should all feel pretty familiar. Except now—you’re not using Cloudflare Tunnels. You’re using bunny.net Magic Containers and pull zones. Not bad.

#### Conclusion

This started as a proof of concept to see if a viable Cloudflare Tunnels alternative could be self-hosted on bunny.net. It’s not perfect yet, but it’s a solid foundation.

I trust bunny.net a lot more not to surprise me with a massive bill—but the actual day-to-day cost of running tunbun is still a bit of a question mark. To find out, I’ll keep the stack running until my free credits run out. That should give a realistic picture of what it costs to route everyday homelab traffic through this setup.

If you like the project, give it a ⭐ on GitHub at [`@alectrocute/tunbun`](https://github.com/alectrocute/tunbun) —and feel free to open a PR.
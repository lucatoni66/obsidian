---
title: "I Fingerprinted 6 Commercial VPNs in 200ms with Open-Source DPI. Here's the Wireshark Data"
source: "https://dev.to/evilork/i-fingerprinted-6-commercial-vpns-in-200ms-with-open-source-dpi-heres-the-wireshark-data-164o"
author:
  - "[[Evilork]]"
published: 2026-04-27
created: 2026-04-27
description: "TL;DR: Modern DPI engines like nDPI detect OpenVPN/WireGuard traffic in under 200ms with 97%... Tagged with security, networking, privacy, opensource."
tags:
  - "clippings vpn"
---
> TL;DR: Modern DPI engines like nDPI detect OpenVPN/WireGuard traffic in under 200ms with 97% accuracy. I tested 6 commercial VPNs. 5 leaked their protocol immediately. Here’s exactly how, with packet captures.

## The Problem Nobody Talks About

When you read “best VPN” articles, they all talk about the same things: speed, server count, no-logs policy, kill switch.

Nobody talks about whether your ISP can **see that you’re using a VPN at all**.

This matters more than you think:

- **Crypto exchanges** (Binance, OKX, Bybit) flag VPN IPs and require additional KYC, sometimes locking funds
- **Payment processors** (Stripe, Adyen) decline transactions from known VPN ranges
- **State-level DPI** in Russia/Iran/China actively profiles VPN users for further monitoring
- **Anti-fraud systems** at banks treat your account as high-risk

If your VPN protocol is detectable - and 95% are - you’re not anonymous, you’re **flagged**.

## The Test Setup

Hardware: Mac M4 Max + remote VPS for traffic mirroring.

Tools:

- **Wireshark** for capture
- **nDPI** (the same library powering most ISP-grade DPI hardware)
- **tshark** for automated parsing
```
# Capture VPN handshake
sudo tshark -i en0 -w vpn_handshake.pcap -a duration:30

# Run through nDPI
ndpiReader -i vpn_handshake.pcap -v 2 | grep -E "VPN|Detected"

# Measure time-to-detection
ndpiReader -i vpn_handshake.pcap -j output.json
jq '.flows[] | {proto: .detected_protocol_name, ms: .first_seen_ms}' output.json
```

I tested each VPN on default settings (what 95% of users use), then ran their “stealth mode” if available.

## The Results

| Provider | Default Protocol | Time to Detect | Stealth Mode | TTD with Stealth |
| --- | --- | --- | --- | --- |
| Provider A (mainstream, top-3 by revenue) | OpenVPN | **180ms** | Obfuscated | 720ms |
| Provider B (mainstream) | WireGuard | **90ms** | None | n/a |
| Provider C (privacy-focused) | IKEv2 | **250ms** | None | n/a |
| Provider D (mainstream) | OpenVPN+XOR | **420ms** | XOR scramble | 420ms (no improvement) |
| Provider E (Asia-focused) | Shadowsocks (legacy) | **1,100ms** | None | n/a |
| Provider F (indie, self-hosted) | **VLESS Reality** | **NOT DETECTED** | n/a | n/a |

**5 out of 6 leaked their VPN signature in under 1.5 seconds.**

The one that didn’t is running VLESS+Reality, a protocol from the XRay project that’s been around since 2023 but mainstream VPNs refuse to adopt.

## What Actually Gets Detected

Here’s a sanitized snippet from the nDPI output for Provider B (WireGuard):

```
{
  "flow_id": 42,
  "src_ip": "10.0.0.5",
  "dst_ip": "[redacted]",
  "detected_protocol": "WireGuard",
  "protocol_category": "VPN",
  "confidence": "DPI",
  "first_seen_ms": 90,
  "fingerprint": {
    "udp_payload_first_byte": "0x01",
    "packet_size_pattern": [148, 92, 92],
    "handshake_signature": "wg_initiation"
  }
}
```

WireGuard’s first packet ALWAYS starts with `0x01` followed by a fixed-size payload. There is no way to disguise this without modifying the protocol itself - and if you modify it, it’s no longer WireGuard.

**This is the dirty secret of “modern” VPN protocols**: they were designed for performance and security against passive eavesdroppers, NOT against active traffic analysis.

## Why VLESS+Reality Wins

Reality doesn’t try to hide that you’re using a VPN. It doesn’t even try to obfuscate the traffic.

It does something smarter: **it pretends to be a real TLS connection to a real high-traffic website**.

```
Client → "Hi server.com, I want to connect"  
Server → [forwards initial handshake to actual server.com]
         [returns real TLS certificate from server.com]
Client → [verifies it's the real server.com]
         [now switches to VPN tunnel under the same connection]
```

To DPI, this looks 100% identical to:

- A user visiting microsoft.com
- A user visiting cloudflare.com
- A user visiting any whitelisted destination

JA3 fingerprint matches Chrome. SNI matches a real CDN. Certificate is real. There’s literally nothing to detect.

## The Code

If you want to test your own VPN, here’s a minimal script:

```
import subprocess
import json
import sys

def test_vpn_fingerprint(interface: str = "en0", duration: int = 30) -> dict:
    """Capture and analyze VPN traffic for protocol fingerprinting."""
    pcap_file = "/tmp/vpn_test.pcap"
    json_file = "/tmp/vpn_test.json"

    # Capture traffic
    subprocess.run([
        "sudo", "tshark", "-i", interface,
        "-w", pcap_file, "-a", f"duration:{duration}"
    ], check=True)

    # Analyze with nDPI
    subprocess.run([
        "ndpiReader", "-i", pcap_file, "-j", json_file
    ], check=True)

    with open(json_file) as f:
        data = json.load(f)

    vpn_flows = [
        flow for flow in data.get("flows", [])
        if "VPN" in flow.get("detected_protocol_name", "")
        or flow.get("detected_protocol_name") in [
            "WireGuard", "OpenVPN", "IPSec", "L2TP"
        ]
    ]

    if vpn_flows:
        print(f"⚠️  VPN detected in {vpn_flows[0]['first_seen_ms']}ms")
        print(f"    Protocol: {vpn_flows[0]['detected_protocol_name']}")
        return {"detected": True, "details": vpn_flows[0]}

    print("✅ No VPN signature detected in capture window")
    return {"detected": False}

if __name__ == "__main__":
    result = test_vpn_fingerprint()
    sys.exit(0 if not result["detected"] else 1)
```

Run this with your VPN connected. If you get the warning - your VPN is detectable.

Install requirements:

```
# macOS
brew install wireshark ndpi

# Ubuntu  
sudo apt install tshark ndpi-utils
```

## What This Means for You as a Developer

If you’re building **anything** that requires real network privacy:

1. **Don’t trust a VPN provider’s marketing**. Test it.
2. **Self-host with VLESS+Reality** if you can. The 3X-UI panel makes it a 10-minute setup.
3. **If you must use commercial**, look for providers explicitly running Reality/Hysteria2/SS-2022. They exist but they’re small.
4. **Consider what you’re protecting against**. If it’s your ISP seeing what site you visit - any VPN works. If it’s your IP getting profiled - you need DPI-resistant protocols.

## The Bigger Picture

The VPN industry is stuck in 2018. They’re optimizing for speed and “no-logs policies” while the threat model has completely shifted.

State actors and large corporations don’t need to read your traffic. They just need to know you’re using a VPN. That alone is enough to:

- Add you to a watchlist
- Trigger anti-fraud at your bank
- Lock your exchange account
- Profile you for further investigation

The protocol matters more than the provider. Stop paying $10/month for legacy protocols dressed up in nice apps.

## Discussion

I’d love to see results from other devs running this script on their VPNs. Drop a comment with:

- Provider name (or anonymized “Provider X”)
- Default protocol
- Time-to-detection in ms

Bonus points if you find a mainstream provider that survives DPI. I haven’t found one.

---

*Code is MIT-licensed, do whatever you want with it. If you reproduce these tests on more providers, I’d love to see a write-up.*[Guardsquare](https://dev.to/guardsquare)Promoted

[![Guardsquare image](https://media2.dev.to/dynamic/image/width=775%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fi.imgur.com%2FMgqS48C.jpeg)](https://hubs.la/Q046c5Q40?bb=262299)

## Mobile App Security Predictions in 2026: How You Can Stay Ahead of Threats and Attacks

The mobile app threat landscape is constantly changing, with attackers continuously evolving techniques. In 2026, staying one step ahead of attackers will be crucial. With Guardsquare, achieve comprehensive mobile app security without compromises.
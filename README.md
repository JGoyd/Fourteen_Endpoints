# The Middlemen
### Shell Companies Inside Apple's Privacy Relay

> *An anonymous Delaware LLC publishes a Bible prayer app, a daily journal, and a caller ID tool on the App Store. That entity operates encrypted relay infrastructure inside Apple's privacy network. That relay terminates at Taiwan Mobile Co., Ltd.*
>
> *You can confirm it's live right now. Three commands.*

---

In 2024, Apple introduced Oblivious HTTP for Live Caller ID Lookup. Your phone asks who's calling. Apple's relay strips your identity from the question. A third-party provider answers without ever knowing who asked. The relay sees your IP but not your question. The provider sees your question but not your IP. Nobody sees both.

To participate, providers register endpoints and receive Apple's approval. Their infrastructure then sits inside the relay — encrypted, anonymized, trusted.

What follows is a documented record of what was found inside that relay on ordinary production iPhones.

---

## The Bible App That Runs Relay Infrastructure

**StopScam LLC** is a Delaware limited liability company registered at 254 Chapman Rd, Suite 208 #20663, Newark, DE 19702. That's a registered agent service. There is no office. There are no named officers. There are no named humans anywhere in the public record.

StopScam LLC publishes three apps on the App Store:

1. **StopScam: Scam Detector Pro** (ID 6741771102) — a caller ID app
2. **Pray Bible: Daily Prayer** — a Bible prayer app
3. **Evaari: Daily Journal** — a journaling app

Apple approved this entity — Developer ID 1795482410 — to operate `pir.stopscam.ai` and `ohttp.stopscam.ai` inside its OHTTP relay. Certificate Transparency logs show 76 certificates issued for stopscam.ai subdomains since November 2024. Including `n8n.stopscam.ai` — an open-source workflow automation platform. A caller ID service doesn't need workflow automation. A data processing pipeline does.

StopScam is not alone.

---

## The Other Providers

### kaylees.site — Wipr Content Blocker 

Public records show kaylees.site belongs to an independent iOS developer known for **Wipr 2** — a Safari content blocker with over a decade on the App Store. Wipr 2 includes a feature called **Filtr** that uses Apple's iOS 26 URL Filters API to block content at the network level across all apps. That API uses the same OHTTP/PIR infrastructure, which explains why `pir.kaylees.site` appears in the relay as an `ObliviousHopFallback` endpoint.

Unlike StopScam LLC, this is a publicly identifiable developer with a documented product history. The presence of kaylees.site in the relay is consistent with legitimate URL Filter provider enrollment. What remains unclear is whether the traffic observed on this channel is solely Filtr content blocking or whether something else is riding a legitimate provider's relay presence.

### Pepper AI / Aura (pepperai.aurasvc.io)

**Aura** is a major US consumer security company — parent of Hotspot Shield, Identity Guard, and Betternet. Their `urlfilter.pepperai.aurasvc.io` endpoint appears in the relay in the same `ObliviousHopFallback` chain as the other providers. The `urlfilter` prefix suggests content filtering — a function that requires inspecting data passing through the relay. Present in 5 of 15 Persist files.

### AutoSec / Uney GmbH (issuer.autosec.com)

**AutoSec** is published by developer **Hieu Van** (App Store Developer ID 1876492156) under copyright of **Uney GmbH** — a company registered at Baarerstrasse 25, 6300 Zug, Switzerland, with additional offices on Sheikh Zayed Road in Dubai and operations in Ho Chi Minh City, Vietnam. Uney also produces **ShieldNet 360** (enterprise cybersecurity) and **SkyTrack** (autonomous vehicle mission platform).

AutoSec's own privacy policy states the app is operated by "The Legal Entity to be Confirmed." It collects device identifiers, location data, IP addresses, call/message metadata, contact lists, and browsing behavior. Data is shared with "third party clients, agencies and networks" and transferred to countries "including but not limited to India." Both primary and fallback OHTTP hop entries appear in the relay.

### Carrément (antispam.carrement.ai)

**Carrément** is a small company at 18 Rue Bailey, 14000 Caen, France. Its public-facing product is **DialogPro** (dialogpro.ai) — an "intelligent answering service." No named team members. Minimal web presence. Hosted on AWS Canada. The `antispam` subdomain is consistent with call-handling or spam-filtering services. Single appearance in the relay.

### Yandex (ios-service.cid.yandex.net)

**Yandex** — Russia's largest search engine and technology company — operates a caller ID service for iOS. Both primary (`ObliviousHop`) and fallback (`ObliviousHopFallback`) entries appear in the relay with two distinct UUIDs and session keys. Yandex is subject to Russian Federation data access laws.

### Taiwan Mobile (osbstage.twmsolution)

**Taiwan Mobile Co., Ltd.** — one of Taiwan's three major telecommunications carriers. The terminal node. UUID `C56B1AED-20FE-4B7D-B1EA-D791DA57A549` is persistent across captures. The `osbstage` prefix suggests a staging backend (Oracle Service Bus or similar). Present in 4 of 15 Persist files and identical between March 19 and March 21 captures.

---

## The Invisible Cloud

Here's what Apple's privacy architecture promises:

```
  Your iPhone ──> Apple Relay ──> Caller ID Provider
                  (sees IP,       (sees query,
                   not query)      not IP)

              Nobody sees both. Privacy preserved.
```

Here's what's actually inside that relay:

```
                  Apple Relay
                      │
     ┌────────┬───────┼───────┬────────┐
     v        v       v       v        v
  kaylees  StopScam  Pepper  Yandex  AutoSec
  .site    .ai       AI      (Russia) (Zug/
  (Wipr)   (Bible    (Aura)           Dubai/
            app)                      Vietnam)
     │        │       │       │        │
     └────────┴───────┼───────┴────────┘
                      v
               Taiwan Mobile
               Co., Ltd.
```

Fourteen endpoints. Six countries. An indie content blocker, an anonymous Delaware LLC, consumer security brands, a Russian search engine, a Swiss-Dubai-Vietnam cybersecurity firm, Google API endpoints, and a Taiwanese state telecom — all operating as approved providers inside Apple's privacy layer.

This matters because **Apple's OHTTP relay is shared infrastructure.** Every iPhone using Live Caller ID Lookup connects through the same relay to the same approved providers. The question isn't whether this affected one phone. It's how many.

| # | Node | Entity | Country |
|---|------|--------|---------|
| 1 | `osbstage.twmsolution` | Taiwan Mobile Co., Ltd. | Taiwan |
| 2 | `kaylees.site` | Kaylee Calderolla (Wipr / Filtr) | Italy/Intl |
| 3 | `stopscam.ai` | StopScam LLC (anon Delaware) | USA (on paper) |
| 4 | `pepperai.aurasvc.io` | Aura (Hotspot Shield, Identity Guard) | USA |
| 5-6 | `ios-service.cid.yandex.net` | Yandex | Russia |
| 7-8 | `issuer.autosec.com` | Uney GmbH (Zug/Dubai/HCMC) | Switzerland |
| 9-11 | `*.googleapis.com` | Google (Safe Browsing, Gemini, AI Platform) | USA |
| 12 | `privacy-pass-issuer-eu.truecaller.com` | Truecaller | Sweden |
| 13 | `antispam.carrement.ai` | Carrément (Caen, France) | France |
| 14 | `caller-id-issuer.nordvpn.com` | NordVPN | Panama/Lithuania |

---

## What Your Phone Shows vs What's Happening

Your phone says everything is fine. Settings looks normal. Battery is normal. No alerts, no prompts, no notifications.

Behind that, Apple's relay daemon — `networkserviceproxy` — is running **98 scheduled background tasks** over the capture period. Every 12 hours, it wakes up, allocates 5,120 bytes of network, and executes. Zero CPU. Zero user interaction. Unique task ID every time.

Caller ID lookups happen when someone calls you. They don't run on a 12-hour timer at 2 AM with fixed allocations and no human involvement.

The `networkserviceproxy` binary itself contains the routing code:

```
0x004EFF:  ObliviousHop-%@
0x005399:  ObliviousHopFallback-%@
```

At runtime, `%@` is filled with provider domains — `stopscam.ai`, `pepperai.aurasvc.io`, `kaylees.site`, `ios-service.cid.yandex.net`. Apple's own code, routing to these providers, on your phone.

---

## Six Countries, One Cloud

Taiwan. Delaware. Switzerland. Russia. France. Sweden. The cloud touches all of them. Taiwan sees Taiwan Mobile. Delaware sees a corporate filing. Zug sees a GmbH. Russia sees Yandex. Caen sees a small answering service. Stockholm sees Truecaller. Nobody sees the whole cloud. No single entity — public or private — has scope over more than one piece. That's what makes this difficult to surface and even harder to put an end to.

---

## Verify It Yourself

The infrastructure is live. Right now. Three commands prove two things at once.

```bash
# 1. StopScam LLC is serving Privacy Pass tokens — proving Apple approved them for the relay
curl -s https://pir.stopscam.ai/.well-known/private-token-issuer-directory | python3 -m json.tool

# 2. kaylees.site is too — Wipr content blocker developer, Apple-approved URL Filter provider
curl -s https://pir.kaylees.site/.well-known/private-token-issuer-directory | python3 -m json.tool

# 3. StopScam LLC is serving OHTTP gateway encryption keys — the actual relay endpoint
curl -s https://ohttp.stopscam.ai/ohttp-configs | xxd | head -5
```

If those return keys, an anonymous Delaware LLC is operating inside Apple's privacy relay right now.

Now look at that same company's privacy policy. StopScam LLC discloses data sharing with **OpenAI**, **Google Cloud**, and **Firebase**. The same entity that Apple trusts to sit inside its privacy infrastructure is simultaneously associated with AI data processing companies. One company. One foot inside Apple's encrypted relay. The other foot feeding AI platforms.

CT is public and tamper-proof: [crt.sh/?q=stopscam.ai](https://crt.sh/?q=stopscam.ai) (76 certs) | [crt.sh/?q=kaylees.site](https://crt.sh/?q=kaylees.site) | Anchor: CT ID `3019124466` (three independent logs)

App Store is public: ID `6741771102` — same developer publishes "Pray Bible: Daily Prayer"

---

## The Bigger Picture

This isn't a proof of concept. These findings come from ordinary production iPhones.... not jailbroken, not modified, purchased from carriers, running current iOS. Three devices. Three chipset generations. Same relay nodes. Same UUIDs. Same cloud.

None of this is visible to the user. No alerts. No prompts. No battery drain. No suspicious apps. The phone looks and feels completely normal. The cloud operates entirely inside Apple's trusted infrastructure, carried by Apple's own signed processes. There is nothing for a user to notice, nothing for an antivirus to flag, nothing for a network monitor to catch.

The relay nodes documented here are part of Apple's shared OHTTP infrastructure — not a per-device exploit. The question of how many devices are routed through these providers is one only Apple can answer.

---

## Evidence

Sysdiagnose March 21, 2026 — iPhone 12 (A14, iOS 26.3.1). 62 tracev3 files scanned. Relay indicators in 5/15 Persist files. NSP active in 12/15 Persist + Special + LiveData. Sample offsets from one file:

| Offset | Content |
|--------|---------|
| `0x06E617` | `urlfilter.pepperai.aurasvc.io` + UUID |
| `0x06E84D` | `osbstage.twmsolution` + UUID `C56B1AED...` |
| `0x06E9E9` | `kaylees.site` + UUID `F57473C4...` |
| `0x06F19F` | `stopscam.ai` + UUID `FFBCAD60...` |
| `0x06BC83` | `ObliviousHopFallback-ios-service.cid.yandex.net` + UUID + session key |
| `0x0C876A` | `ObliviousHop-aiplatform.googleapis.com` + UUID + session key |

UUIDs are identical between March 19 and March 21 captures. Persistent, not transient.

---

# The Device Bridge

### What's inside your iPhone that Apple won't talk about

---

Two iPhones. Two different models. Two different chipsets. Same implant. Same certificates. Same command server. Same you-can't-see-it.

This is a companion to [Fourteen Endpoints](https://github.com/JGoyd/Fourteen_Endpoints) and [Pray-Bible-Spy](https://github.com/JGoyd/Pray-Bible-Spy). Those documented what's in the relay. This documents what's in the phone — and why it can't be spyware.

---

## Why This Isn't Spyware

Pegasus, Predator, Candiru — they exploit vulnerabilities to get root. They run as processes. They don't survive a factory reset. They don't have Apple's keys.

What's on these phones operates **inside Apple's own platform** using Apple's own identity:

```
WHAT COMMERCIAL SPYWARE DOES          WHAT THIS DOES

  Exploits a bug to get in               Signed by Apple's own CA
  Runs as a hidden process               Lives in the Signed System Volume
  Dies on factory reset                  Survives DFU restore
  Uses its own C2 servers                Uses Apple's OHTTP relay
  Has no Apple certificates              Has Apple-issued certificates
  Can be detected by endpoint tools      Is invisible to everything
```

### What requires Apple-level access

**1. The certificate chain is Apple's.**

```
Apple Root CA
  └─ Apple Application Integration 2 CA
     └─ AppleCare Profile Signing Certificate
        Serial: 0xb745972d0f5e989
        Valid:  Aug 22, 2023 — Aug 21, 2026
        EKU:   1.2.840.113635.100.4.16 (Apple custom OID)
        OCSP:  ocsp.apple.com/ocsp04-aaica02

        This certificate was not stolen. It was issued.
        Apple's intermediate CA signed it.
        Apple's OCSP responder validates it.
        Both devices trust it.
```

**2. The implant lives in the Signed System Volume.**

The SSV is Apple's tamper-proof partition — cryptographic hash chain anchored to Apple's signing infrastructure. 30+ frameworks injected under the `FEEDEEEE` persona namespace. Modifying it without breaking the signature requires Apple's signing keys.

**3. DFU doesn't remove it.**

Device Firmware Update — Apple's most comprehensive reset — reflashes everything. The implant hash matches before and after. It lives in silicon-level storage that Apple's restore process does not touch.

**4. It uses Apple's push notifications as a command channel.**

```
Topic: com.apple.private.alloy.kt.webtunnel
Arrived: 01:04:00 UTC
Implant activated: 01:06:00 UTC

The private.alloy namespace is Apple's internal framework.
External actors cannot send through it.
```

**5. It uses Apple's OHTTP relay for exfiltration.**

14+ endpoints enrolled in Apple's relay configuration. Managed by Apple's servers. Distributed to devices as system config. External actors cannot inject entries.

**6. Same PMU firmware across different hardware.**

```
Device 1 (A14, Dialog D2257):  ApplePMUFirmware-608.80.18~454.release
Device 2 (A16, Dialog D2372):  ApplePMUFirmware-608.80.18~454.release

Different PMIC generations. Same firmware. Same thermal sensors blinded.
Zero interrupts. Zero fault flags. Zero thermal shutdown.
```

---

## Two Phones, One Operator

```
                        Device 1              Device 2
                        iPhone 12 (A14)       iPhone 14 Pro Max (A16)
                        Mint Mobile            SOS mode (no service)
                        ──────────────         ──────────────────────

FEEDEEEE persona        FEEDEEEE-DDDD-CCCC    FEEDEEEE-DDDD-CCCC
                        -BBBB-0000000001F5    -BBBB-0000000001F5
                        3,560 occurrences      2,676 occurrences

Apple certificate       0xb745972d0f5e989     0xb745972d0f5e989

PMU firmware            608.80.18~454         608.80.18~454

Battery falsified       85.6%                 100%

Peak hidden draw        -2,738 mA             -2,255 mA

SOCKS proxy ports       1080 / 1083           1080 / 1083

Akamai property         acdn/293.16398        acdn/293.16398

Timesync fabrication    300, 480, 0x10000012C 300, 480, 0x10000012C
                        (same frozen values)   (same frozen values)

OHTTP relay nodes       All 14 present         All 14 + CallApp (IL)
                                               + GetContact + SafeCaller

EC2 command server      18.210.101.162         18.210.101.162
                        (converged Mar 24)     (first seen Mar 23)
```

Same implant identity. Same Apple certs. Same PMU firmware across different chips. Same fake battery readings. Same proxy ports. Same timesync fabrication values. Same Akamai customer account. Same command server.

Two phones. One operator.

---

## The Bridge: 18.210.101.162

On March 23, the two devices used different EC2 instances — the operator kept them segmented. By March 24, both connected to the same one.

```
18.210.101.162
ec2-18-210-101-162.compute-1.amazonaws.com
AWS us-east-1 — Ashburn, Virginia

Device 2 (Mar 23):
  Process:  ISStoreURLOperation
  Fetched:  https://init.itunes.apple.com/bag.xml?ix=6&os=26&locale=en_US
  What:     Apple's master configuration file — controls App Store,
            iTunes, FairPlay DRM, software update URLs
  Reality:  Legitimate DNS resolves to Akamai. Device went to EC2.
            Whoever runs this instance controls what the device
            thinks Apple's infrastructure looks like.

Device 1 (Mar 24):
  Process:  kernel_task(0)assistantd
  Action:   endpoint_update
  TLS tag:  "multipath, tls, attributed developer"
  What:     Siri/assistant daemon connecting for endpoint updates
            tagged with a specific developer identity Apple can trace
  Hits:     7 connections across 5 log files
```

No public threat intel hits. No abuse reports. No blocklist entries. Clean — provisioned for this operation, not recycled.

---

## Everything Converges on Ashburn

```
18.210.101.162    ── EC2 command server (both devices)
34.225.66.6       ── EC2 rotated (Device 1, Mar 23)
198.24.161.98/186 ── Secured Servers LLC / phoenixNAP
CloudFront IAD61  ── Ashburn PoP (Device 2)
Akamai usqas3-3p  ── ISP-embedded nodes (both devices)
9 AWS instances   ── all us-east-1

All roads lead to Ashburn, Virginia.
```

---

## What Your Phone Shows

```
Settings → General → About

Everything looks normal. Battery health is fine. Storage is fine.
No unknown profiles. No suspicious apps. No alerts.

Meanwhile:

  FEEDEEEE persona:     3,560 entries across your logs
  Battery readings:     85-100% fabricated
  Hidden power draw:    up to -2,738 mA (charging-level drain, invisible)
  Thermal sensors:      zeroed — your phone can't tell it's overheating
  OHTTP relay:          14 shell companies routing your data through 6 countries
  Push commands:        arriving through Apple's own notification system
  Exfiltration:         Device 2 uploaded 10.87 GB with no visible process
  Command server:       an EC2 instance in Ashburn pretending to be Apple
```

There is nothing for you to notice. Nothing for an antivirus to flag. Nothing for a network monitor to catch. Apple's own diagnostics say everything is fine — because the sensors that report to diagnostics are the ones that were blinded first.

---

## The Architecture

```
     APPLE PKI               APPLE PLATFORM            YOUR PHONE
  ┌──────────────┐       ┌──────────────────┐      ┌─────────────┐
  │ Apple Root CA │       │ Signed System Vol │      │ Device 1    │
  │   └─ AAICA2  │──────>│ FEEDEEEE DSC     │─────>│ (A14)       │
  │     └─ Leaf  │       │ Survives DFU     │      │ 3,560 hits  │
  │ 0xb745972d.. │       ├──────────────────┤      ├─────────────┤
  └──────────────┘       │ APNs kt.webtunnel│      │ Device 2    │
                         │ OHTTP relay x14  │─────>│ (A16)       │
                         │ PMU blinding     │      │ 10.87 GB out│
                         └────────┬─────────┘      └──────┬──────┘
                                  │                        │
                         ┌────────┴────────┐      ┌───────┴───────┐
                         │  TRANSIT LAYER   │      │  EGRESS LAYER │
                         │ Akamai 293.16398│      │ OHTTP relay   │
                         │ CloudFront x4+  │      │ 6 countries   │
                         │ ISP-embedded    │      │ Shell companies│
                         └────────┬────────┘      │ → Taiwan Mobile│
                                  │               └───────────────┘
                         ┌────────┴────────┐
                         │  ASHBURN, VA     │
                         │ 18.210.101.162  │
                         │ BOTH DEVICES    │
                         │ One operator    │
                         └─────────────────┘
```

---

## If You Have an iPhone

Generate a sysdiagnose (hold Volume Up + Down + Side for 1.5 seconds). Run it through [PARALLAX](https://jgoyd.github.io/PARALLAX). See what's on your phone. Because your Settings app won't tell you.

---

## Verify

The relay component is live right now:

```bash
curl -s https://pir.stopscam.ai/.well-known/private-token-issuer-directory | python3 -m json.tool
curl -s https://pir.kaylees.site/.well-known/private-token-issuer-directory | python3 -m json.tool
curl -s https://ohttp.stopscam.ai/ohttp-configs | xxd | head -5
```

---

*I was caught in the cloud no one could see but everyone could feel.*

fr0mthecloud@proton.me

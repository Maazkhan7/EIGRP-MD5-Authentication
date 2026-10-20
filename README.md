<div align="center">

# 🔐 EIGRP MD5 Authentication

### Secure Neighbor Adjacency · Key Chains · Routing Protocol Security — Cisco EIGRP AS 1

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Protocol](https://img.shields.io/badge/Protocol-EIGRP-blue?style=for-the-badge)
![Security](https://img.shields.io/badge/Security-MD5%20Authentication-critical?style=for-the-badge)
![AS](https://img.shields.io/badge/AS-1-orange?style=for-the-badge)
![Routers](https://img.shields.io/badge/Routers-2911-green?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA%20%2F%20CCNP-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

</div>

---

## 📌 Project Overview

By default, **any router** that sends EIGRP packets on a shared link with the right AS number can form a neighbor adjacency and start injecting routes. That is a real attack surface: a rogue or misconfigured router can poison the routing table.

This lab secures an EIGRP adjacency with **MD5 authentication** using a **key chain**. Two Cisco 2911 routers (**R1**, **R2**) are connected by a serial link (**1.1.1.0/24**), each with its own LAN. After authentication is enabled:

- Every EIGRP packet carries an **MD5 digest** built from the packet contents and a **shared secret key**
- A neighbor **accepts only packets with a valid digest**
- Routers with a missing or wrong key **cannot form an adjacency**

> 🎯 **What the lab proves:** when R1 turned on MD5 first, the existing adjacency **dropped** (`authentication mode changed`). It came back (`new adjacency`) **only after R2 was configured with a matching mode, key ID and key string.**

---

## 🎯 Objectives

- ✅ Create a **key chain** with a key ID and key string on both routers
- ✅ Enable **MD5 authentication** for EIGRP AS 1 on the serial interface
- ✅ Bind the interface to the key chain
- ✅ Observe the adjacency **go down** when only one side is authenticated
- ✅ Observe the adjacency **come back up** when both sides match
- ✅ Verify the settings in the **running-config**
- ✅ Understand what MD5 authentication protects (and what it does not)

---

## 🗺️ Network Topology

```
   192.168.1.0/24                                           192.168.2.0/24

PC1 ─┐                       1.1.1.0/24                           ┌─ PC4
PC2 ─┼─ Switch0 ── Gi0/0 ┌──────┐ Se0/0/0 ═══════ Se0/0/0 ┌──────┐ Gi0/0 ── Switch1 ─┼─ PC5
PC3 ─┘  (2950)    .1     │  R1  │ .1     🔐 MD5      .2   │  R2  │  .1    (2950)      └─ PC6
                        │ 2911 │                          │ 2911 │
                        └──────┘                          └──────┘
                   EIGRP AS 1 · key chain "khan"     EIGRP AS 1 · key chain "khan"
```

> 📷 Add your screenshot: `![Topology](images/topology.png)`

---

## 📋 IP Addressing Table

| Device | Interface | IP Address | Mask | Connected To |
|:------:|:---------:|:----------:|:----:|:------------:|
| **R1** | Gi0/0 | 192.168.1.1 | 255.255.255.0 | Switch0 (LAN 1) |
| **R1** | Se0/0/0 | 1.1.1.1 | 255.255.255.0 | R2 Se0/0/0 |
| **R2** | Gi0/0 | 192.168.2.1 | 255.255.255.0 | Switch1 (LAN 2) |
| **R2** | Se0/0/0 | 1.1.1.2 | 255.255.255.0 | R1 Se0/0/0 |
| PC1 / PC2 / PC3 | NIC | 192.168.1.2 / .3 / .4 | 255.255.255.0 | Switch0 |
| PC4 / PC5 / PC6 | NIC | 192.168.2.2 / .3 / .4 | 255.255.255.0 | Switch1 |

---

## 🔑 Authentication Parameters

| Parameter | Value | Must match on both routers? |
|:----------|:-----:|:---------------------------:|
| EIGRP AS number | **1** | ✅ Yes |
| Authentication mode | **md5** | ✅ Yes |
| Key chain name | `khan` | ❌ No (locally significant) |
| Key ID | **1** | ✅ **Yes** |
| Key string | `cisco` | ✅ **Yes** |
| Applied on | Serial0/0/0 | Each side of the link |

---

## ⚙️ Configuration

> 💡 Basic EIGRP AS 1 (network statements) was already running — this lab **adds security on top of it**.

### 🔹 Step 1 — Create the key chain (R1 **and** R2)

```bash
key chain khan
 key 1
  key-string cisco
```

### 🔹 Step 2 — Enable MD5 and bind the key chain on the interface (R1 **and** R2)

```bash
interface Serial0/0/0
 ip authentication mode eigrp 1 md5
 ip authentication key-chain eigrp 1 khan
```

### 🔹 Step 3 — Save

```bash
do write memory
```

### 🔹 Full command flow as typed in the lab

**R1**
```
R1(config)# key chain khan
R1(config-keychain)# key 1
R1(config-keychain-key)# key-string cisco
R1(config-keychain-key)# exit
R1(config-keychain)# do wr
R1(config-keychain)# exit
R1(config)# interface serial 0/0/0
R1(config-if)# ip authentication mode eigrp 1 md5
%DUAL-5-NBRCHANGE: IP-EIGRP(0) 1: Neighbor 1.1.1.2 (Serial0/0/0) is down: authentication mode changed
R1(config-if)# ip authentication key-chain eigrp 1 khan
R1(config-if)# do wr
```

**R2**
```
R2(config)# key chain khan
R2(config-keychain)# key 1
R2(config-keychain-key)# key-string cisco
R2(config-keychain-key)# exit
R2(config)# interface serial 0/0/0
R2(config-if)# ip authentication mode eigrp 1 md5
R2(config-if)# ip authentication key-chain eigrp 1 khan
%DUAL-5-NBRCHANGE: IP-EIGRP 1: Neighbor 1.1.1.1 (Serial0/0/0) is up: new adjacency
R2(config-if)# do wr
```

---

## ✅ Verification

### 1️⃣ Adjacency behavior (the key evidence)

| Step | Event | Log message |
|:----:|:------|:------------|
| 1 | R1 enables MD5 → R2 still has none | `Neighbor 1.1.1.2 (Serial0/0/0) is down: authentication mode changed` ❌ |
| 2 | R2 enables MD5 with the **same key** | `Neighbor 1.1.1.1 (Serial0/0/0) is up: new adjacency` ✅ |

🧠 **Meaning:** R1 now signs every packet with MD5, R2 (without authentication) cannot validate them, so the adjacency is torn down. Once R2 uses the same key ID and key string, packets validate and the neighbor relationship is re-established.

### 2️⃣ Running-config (R1) — `show running-config`

```
key chain khan
 key 1
  key-string cisco
!
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
!
interface Serial0/0/0
 ip address 1.1.1.1 255.255.255.0
 ip authentication mode eigrp 1 md5                  ◄── 🔐 MD5 enabled for AS 1
 ip authentication key-chain eigrp 1 khan            ◄── 🔑 bound to key chain "khan"
 clock rate 64000
```

### 3️⃣ Recommended extra captures (to complete the portfolio)

```bash
show ip eigrp neighbors              # adjacency is up again after authentication
show ip route eigrp                  # remote LAN learned (D 192.168.2.0/24 ...)
show key chain                       # key 1 and key-string
show ip eigrp interfaces detail      # authentication mode on Se0/0/0
ping 192.168.2.2                     # end-to-end connectivity after securing EIGRP
```

---

## 🔍 How EIGRP MD5 Authentication Works

```
 R1                                                        R2
 ┌───────────────────────────┐                ┌───────────────────────────┐
 │ EIGRP packet (Hello/Update)│                │                           │
 │        +                   │   packet +     │ 1. Read key ID from packet│
 │ Key ID 1 + key-string      │ ─────────────► │ 2. Look up key 1 locally  │
 │        ↓                   │   MD5 digest   │ 3. Recompute MD5 digest   │
 │ MD5 digest computed        │                │ 4. Compare digests        │
 └───────────────────────────┘                │    ✅ equal → accept       │
                                               │    ❌ different → drop     │
                                               └───────────────────────────┘
```

- The **key string is never sent** — only the digest is
- All EIGRP packet types are protected: **Hello, Update, Query, Reply, Ack**
- A wrong key means packets are **silently discarded** → **no adjacency**

### 🛡️ What it protects against

| Threat | Protected? |
|:-------|:----------:|
| Rogue router forming an adjacency | ✅ |
| Fake route injection / routing table poisoning | ✅ |
| Accidental neighbor from a misconfigured device | ✅ |
| Packet tampering in transit | ✅ |
| **Eavesdropping** on routing updates | ❌ MD5 authenticates — it does **not encrypt** |

---

## 🧰 Troubleshooting Guide

| Symptom | Likely Cause | Fix |
|:--------|:-------------|:----|
| Neighbor down: `authentication mode changed` | One side enabled MD5, the other did not | Enable the same mode on both sides |
| Neighbor never comes up after config | **Key string mismatch** | Re-enter the same `key-string` |
| Neighbor never comes up | **Key ID mismatch** (`key 1` vs `key 2`) | Use the same key ID |
| Neighbor never comes up | Key chain not bound to the interface | `ip authentication key-chain eigrp 1 <name>` |
| Neighbor never comes up | Wrong **AS number** in the authentication commands | Use the real AS (here **1**) |
| Works, then fails later | Key lifetime expired | Check `accept-lifetime` / `send-lifetime` and device clocks |
| `% Invalid input detected` | Typo / wrong mode | Re-enter the command in the right config mode |

> 💡 The **key chain name** can differ between routers (it is only a local label). The **key ID and key string** must match.

---

## 🚀 Advanced Topics

### 🔄 Key rotation with lifetimes

Key chains allow **graceful key rollover** — two keys overlap so the adjacency never drops:

```bash
key chain khan
 key 1
  key-string cisco
  accept-lifetime 00:00:00 Jan 1 2026 00:00:00 Jan 1 2027
  send-lifetime   00:00:00 Jan 1 2026 00:00:00 Jan 1 2027
 key 2
  key-string NewStr0ngKey!
  accept-lifetime 00:00:00 Dec 1 2026 infinite
  send-lifetime   00:00:00 Jan 1 2027 infinite
```
> ⏰ Lifetimes depend on correct **device time** (use NTP).

### 🔒 Hardening ideas

- Use a **long, complex key string** — `cisco` is fine for a lab but weak in production
- `service password-encryption` can obfuscate key strings in the running-config
- Add `passive-interface` on LAN-facing interfaces so EIGRP packets aren't sent to PCs
- On newer IOS, **EIGRP named mode** supports stronger **HMAC-SHA-256** authentication instead of MD5

---

## 🎓 Key Takeaways

1. 🔸 EIGRP without authentication trusts **any** router in the same AS on the link
2. 🔸 MD5 authentication needs **three things**: key chain → mode `md5` → key chain bound to the **interface**
3. 🔸 Authentication is configured **per interface**, not globally
4. 🔸 **Key ID and key string must match**; the key chain *name* is local
5. 🔸 Enabling MD5 on **one side only** drops the adjacency — a handy way to test it
6. 🔸 The adjacency returns automatically once both sides match
7. 🔸 MD5 gives **authenticity and integrity**, **not confidentiality**
8. 🔸 Always verify with `show ip eigrp neighbors` and a ping after securing a link

---

## 🛠️ Useful Commands

```bash
show key chain
show ip eigrp neighbors
show ip eigrp interfaces detail
show ip route eigrp
show ip protocols
show running-config interface serial 0/0/0
debug eigrp packets                  # lab only — shows authentication failures
```

---

## 🧪 Challenges

1. Change the key string on **R2 only** — does the neighbor drop? Which log or debug shows why?
2. Use **different key IDs** (1 on R1, 2 on R2) — what happens?
3. Rename the key chain on R2 (e.g., `secure`) — does the adjacency survive?
4. Add a **second key** with lifetimes and test a smooth rollover
5. Add a third router **without** authentication — can it become a neighbor?

---

## 📁 Repository Structure

```
📦 EIGRP-MD5-Authentication
 ┣ 📜 README.md
 ┣ 📂 images
 ┃ ┣ 🖼️ topology.png
 ┃ ┣ 🖼️ 01-r1-key-chain.png
 ┃ ┣ 🖼️ 02-r1-md5-adjacency-down.png
 ┃ ┣ 🖼️ 03-r2-md5-adjacency-up.png
 ┃ ┗ 🖼️ 04-r1-running-config.png
 ┗ 📂 lab-file
   ┗ 📦 EIGRP-MD5-Authentication.pkt
```

---

## 🔗 Series Navigation

⏮️ **Start:** EIGRP Successor – Single Path
⬅️ **Previous:** EIGRP Unequal-Cost Load Balancing & Variance

`EIGRP` · `MD5` · `Authentication` · `Key Chain` · `Routing Security` · `Neighbor Adjacency` · `Network Hardening`

---

## 👨‍💻 Author

**Maaz Khan**
CCNA-Certified Network & NOC Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-maazkhanms-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/maazkhanms)
[![GitHub](https://img.shields.io/badge/GitHub-maazkhanms-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/maazkhanms)

---

<div align="center">

⭐ **If this lab helped you, star the repository!** ⭐

*Part of my ongoing Cisco Packet Tracer lab series — routing, switching, security & more.*

</div>

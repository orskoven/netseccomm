# 📡 Network Security Architecture Report – Fortune 10 Compliance

**Author**: ChatGPT — Cybersecurity Architect  
**Audience**: Enterprise Network Security Teams, SOC Analysts, Architecture Review Boards  
**Confidentiality**: Internal Use Only  
**Last Reviewed**: 2025-05-02

---

## 🔐 Executive Summary

This report outlines a zero-trust, stateful firewall policy design and ACL enforcement structure for a midsize organization, based on Fortune 10 best practices. The design enforces layered segmentation, principle of least privilege, and full visibility into connection states for critical assets and business units.

---
```mermaid
flowchart LR
    %% ================= Zones =================
    subgraph INTERNET [🌐 Internet Zone]
        EXT_USERS[External Users]
        CDN[CDN / Update Servers]
        DNS[8.8.8.8 (Public DNS)]
    end

    subgraph DMZ [🔒 DMZ Zone]
        WEB1[WEB1 - Public Web Server]
    end

    subgraph INTERNAL [🏢 Internal Network]
        DB01[DB01 - App DB (MySQL)]
        DB02[DB02 - Data Warehouse]
        WEB2[WEB2 - Internal Tools]
        FIL1[FIL1 - File Server]
    end

    subgraph DEV [🧪 Development Zone]
        DEV_WEB1[Dev-WEB1 (Clone)]
        DEV_DB01[Dev-DB01 (Clone)]
        DEV_WEB2[Dev-WEB2 (Clone)]
        DEV_DB02[Dev-DB02 (Clone)]
        DEV_TEAM[Dev Team (10 Hosts)]
    end

    subgraph USERS [👥 User VLANs]
        SALES[Sales Team (50 Hosts)]
        SUPPORT[Support Team (10 Hosts)]
    end

    subgraph WIFI [📶 WiFi Guest]
        WIFI_USERS[WiFi Users]
    end

    %% =============== Trusted Flows ===================
    EXT_USERS -->|HTTP/HTTPS| WEB1
    WEB1 -->|TCP 3306| DB01
    SALES -->|SMB / SQL| FIL1
    SALES -->|SQL| DB02
    SUPPORT -->|HTTPS Tools| WEB2

    DEV_TEAM --> DEV_WEB1
    DEV_TEAM --> DEV_DB01
    DEV_TEAM --> DEV_WEB2
    DEV_TEAM --> DEV_DB02
    DEV_TEAM --> WEB1
    DEV_TEAM --> DB01
    DEV_TEAM --> WEB2
    DEV_TEAM --> DB02
    DEV_TEAM --> FIL1

    FIL1 --> DB02

    WIFI_USERS -->|HTTP/HTTPS| CDN
    WIFI_USERS -->|DNS| DNS

    %% =============== Denied/Restricted ================
    WIFI_USERS -.x DB01
    WIFI_USERS -.x DB02
    WIFI_USERS -.x WEB2
    WIFI_USERS -.x FIL1
    WIFI_USERS -.x SALES
    WIFI_USERS -.x SUPPORT

    %% =============== Firewall Labels =================
    INTERNET -. FW1 -. DMZ
    DMZ -. FW2 -. INTERNAL
    DEV -. FW3 -. INTERNAL
    WIFI -. FW4 -. INTERNET

    %% =============== Styles =================
    classDef zone fill:#f9f9f9,stroke:#333,stroke-width:1.5px;
    class INTERNET,DMZ,INTERNAL,DEV,USERS,WIFI zone;


```
___

## 🧱 Network Components

| System ID | Purpose                          | Zone      |
|-----------|----------------------------------|-----------|
| `WEB1`    | Public web server (DMZ)          | DMZ       |
| `DB01`    | Database (used by WEB1)          | Internal  |
| `WEB2`    | Internal web tools (CRM, etc.)   | Internal  |
| `DB02`    | Internal data warehouse          | Internal  |
| `FIL1`    | File server                      | Internal  |
| `Sales`   | 50 users, file + DB02 access      | User VLAN |
| `Support` | 10 users, access to WEB2          | User VLAN |
| `Dev`     | 10 users, access to all + clones  | Dev VLAN  |
| `WiFi`    | Guest internet-only               | WiFi VLAN |

---

## 🛡️ Firewall Policy Matrix

| **Politik**                                                                                   | **Firewall-indstilling**                                                                         |
|-----------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| Ingen adgang til internettet fra interne servere (WEB2, DB01, DB02, FIL1)                     | Drop alle udgående pakker fra IP’erne for WEB2, DB01, DB02, og FIL1 til destination-port 80/443  |
| Kun WEB1-serveren må modtage forbindelser udefra på port 80 og 443                            | Drop alle indgående TCP SYN-pakker til enhver IP undtagen WEB1 (port 80 og 443)                  |
| Adgang til DB01 er kun tilladt fra WEB1                                                       | Drop alle TCP-pakker til DB01 undtagen dem, der kommer fra WEB1                                  |
| Adgang til DB02 er kun tilladt fra FIL1 og salgsteamet                                        | Drop alle TCP-pakker til DB02 undtagen dem, der kommer fra FIL1 og salgsteamets subnet           |
| Intern WEB2 må kun tilgås af teknisk support og udviklingsteam                                | Drop alle TCP-pakker til WEB2 undtagen dem, der kommer fra support- og udviklings-subnet         |
| Udviklingsteam har adgang til alt                                                             | Tillad al trafik fra udviklings-subnet                                                           |
| Bloker adgangen til interne systemer fra WiFi-netværket                                       | Drop al trafik fra WiFi-subnet til interne IP-ranges                                             |
| WiFi-brugere må kun tilgå internettet                                                         | Tillad trafik fra WiFi-subnet til port 80/443, drop alle andre forbindelser                      |
| Forhindre uautoriseret filadgang                                                              | Drop al trafik til FIL1 undtagen fra salgsteam og udviklings-subnet                              |
| Forhindre web-radio/båndbredde-misbrug                                                        | Drop alle indgående UDP-pakker undtagen DNS (port 53)                                            |
| Forhindre smurf DoS-angreb                                                                    | Drop alle udgående ICMP echo-request pakker til broadcast-adresser (fx 10.x.x.255)               |
| Forhindre netværks-traceroute                                                                 | Drop alle udgående ICMP TTL expired-meddelelser (ICMP type 11)                                   |
| Drop ukorrekt oprettede TCP-forbindelser (uden 3-vejs handshake)                              | Drop TCP-pakker med uventede flagkombinationer (fx SYN+FIN eller uden ACK/SYN)                   |
| Split-routing per interface - DMZ (WEB1), Intern (WEB2/DB/FIL), WiFi (gæst)                   | Anvend forskellige filtreringsregler pr. interface – DMZ tillader kun HTTP/HTTPS ind/ud til WEB1 |
| Begræns eksterne forbindelser kun til nødvendige destinationer (f.eks. CDN, software updates) | Opsæt whitelist af tilladte eksterne destinationer via DNS eller IP-lister                       |
| Intern adgang til DNS, NTP og centrale tjenester kun via specifikke interne servere           | Drop pakker til DNS/NTP udefra undtagen via autoriserede proxy/DNS/NTP-servere                   |

---

## 📘 Stateful Connection Table (Live Tracking Example)

| **source address** | **dest address**     | **source port** | **dest port** |
|--------------------|----------------------|------------------|----------------|
| 10.10.10.15        | 130.207.244.203      | 55233            | 80             |
| 10.10.10.22        | 10.10.90.10          | 55844            | 3306           |
| 10.10.20.12        | 10.10.90.20          | 50533            | 1433           |
| 10.10.20.18        | 10.10.50.5           | 49411            | 445            |
| 10.10.30.9         | 10.10.60.2           | 51345            | 8080           |
| 10.10.30.7         | 130.207.244.203      | 54678            | 443            |
| 10.10.70.15        | 8.8.8.8              | 60122            | 53             |
| 10.10.10.25        | 199.1.205.23         | 52389            | 443            |
| 10.10.10.17        | 10.10.10.30          | 59331            | 80             |
| 10.10.10.21        | 10.10.10.31          | 59001            | 3306           |

---

## 🔒 Access Control List (ACL) – Stateful Filter

| **action** | **source address** | **dest address**      | **protocol** | **source port** | **dest port** | **flag bit check** | **conxion** |
|------------|--------------------|------------------------|--------------|------------------|----------------|--------------------|-------------|
| allow      | 10.10.10.0/24      | 130.207.244.203        | TCP          | > 1023           | 80             | any                |             |
| allow      | 130.207.244.203    | 10.10.10.0/24          | TCP          | 80               | > 1023         | ACK                | X           |
| allow      | 10.10.10.0/24      | 199.1.205.0/24         | TCP          | > 1023           | 443            | any                |             |
| allow      | 199.1.205.0/24     | 10.10.10.0/24          | TCP          | 443              | > 1023         | ACK                | X           |
| allow      | 10.10.10.0/24      | 10.10.90.0/24          | TCP          | > 1023           | 3306           | any                |             |
| allow      | 10.10.90.0/24      | 10.10.10.0/24          | TCP          | 3306             | > 1023         | ACK                | X           |
| allow      | 10.10.20.0/24      | 10.10.90.20            | TCP          | > 1023           | 1433           | any                |             |
| allow      | 10.10.90.20        | 10.10.20.0/24          | TCP          | 1433             | > 1023         | ACK                | X           |
| allow      | 10.10.20.0/24      | 10.10.50.5             | TCP          | > 1023           | 445            | any                |             |
| allow      | 10.10.50.5         | 10.10.20.0/24          | TCP          | 445              | > 1023         | ACK                | X           |
| allow      | 10.10.70.0/24      | 8.8.8.8                | UDP          | > 1023           | 53             | —                  |             |
| allow      | 8.8.8.8            | 10.10.70.0/24          | UDP          | 53               | > 1023         | —                  | X           |
| deny       | all                | all                    | all          | all              | all            | all                |             |

---

## ✅ Summary of Security Posture

- **Segmentation**: Enforced across DMZ, internal, development, and guest zones  
- **Stateful Inspection**: Full connection tracking with ACK validation  
- **Least Privilege**: Scoped access by role, port, and application logic  
- **Zero Trust**: WiFi and internet access scoped, all internal-to-internet access denied for sensitive servers  
- **Audit-ready**: Policies traceable to source system, team, or zone  
- **Resilience**: Anti-spoofing, DoS suppression, traceroute protection included

---

## 📎 Recommendations

1. Deploy log forwarding to SIEM for all dropped packets
2. Enable alerting on TCP flag anomalies (e.g. XMAS, NULL scans)
3. Regularly audit firewall rules for scope creep
4. Run simulated breach exercises on WiFi and Dev zones quarterly

---

> This document is maintained under ISO/IEC 27001 and NIST SP 800-41 compliance guidelines.

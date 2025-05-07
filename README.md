# 📡 Network Security Architecture Report

**Author**: Liban, Youssef, Ferhat, Simon 
**Audience**: Enterprise Network Security Teams, SOC Analysts, Architecture Review Boards  
**Confidentiality**: Internal Use Only  
**Last Reviewed**: 2025-05-02

---
## Network Architecture



___


## Introduction

Dagligt bliver kommuner og virksomheder i Danmark ramt af ondsindet angreb mod deres netværk.
Netværk kan designes, så de kan lukke ned for ondsindet traffic og kun tillade bestemt trafik på nettet. 

## Subnetting 

For at vi kan skræddersy netværksregler, der kun tillader en bestemt "sikker" trafik, indeler vi netværket i subnets.

## Opdeling af subnets

Subnets kan gøre vores netværk mere sikkert og nemmere at bruge.
Ved at indele et netværk i subnets kan vi åbne trafikken for godkendte brugere, afhæningt af IPv4 addressen, og blokerer adgangen for alle andre.


### Udvælgelse af prefix og CIDR

IP addressen danner rammen for al kommunikation via internettet. Uden en IP addresse kan vi ikke modtage eller sende pakker, på tværs af de forbunde enheder på netværket. 

IP addresser findes i 2 udgaver hhv. IPv4 (den "gamle" og korte) og IPv6 ("den nye" og længere).

IP addresser kan anvendes af både de "gode" og "onde".

### prefix og CIDR

For at vi kan skille de gode fra de onde, anvender vi subnets.

Vi udnytter IPv4 addressens bits og CIDR notation.

### Metode




I bestemmelsen af IPv4 addresser og CIDR notation for subnettet, bruger vi virksomhedens logiske afdelinger hhv.: 
- salg
- teknisksupport
- udvikling
- gæster



Vi tildeler de enkelte afdelinger deres egen

Disse subnets, er bestemt på baggrund af forventet antal brugerer på nettet.


Vi anvender FIGUR 1, hvorfra vi har valgt følgende

---

### 📊 IP Subnetting Table (CIDR to Hosts) FIGUR 1

| CIDR Notation | Subnet Mask     | # of Hosts (Usable) | IP Range Size | Notes                       |
| ------------- | --------------- | ------------------- | ------------- | --------------------------- |
| /32           | 255.255.255.255 | 1                   | 1             | Single host (loopback, etc) |
| /31           | 255.255.255.254 | 2 (point-to-point)  | 2             | Special use for P2P links   |
| /30           | 255.255.255.252 | 2                   | 4             | Small subnets               |
| /29           | 255.255.255.248 | 6                   | 8             |                             |
| /28           | 255.255.255.240 | 14                  | 16            |                             |
| /27           | 255.255.255.224 | 30                  | 32            |                             |
| /26           | 255.255.255.192 | 62                  | 64            |                             |
| /25           | 255.255.255.128 | 126                 | 128           |                             |
| /24           | 255.255.255.0   | 254                 | 256           | Class C size                |
| /23           | 255.255.254.0   | 510                 | 512           |                             |
| /22           | 255.255.252.0   | 1022                | 1024          |                             |
| /21           | 255.255.248.0   | 2046                | 2048          |                             |
| /20           | 255.255.240.0   | 4094                | 4096          |                             |
| /19           | 255.255.224.0   | 8190                | 8192          |                             |
| /18           | 255.255.192.0   | 16,382              | 16,384        |                             |
| /17           | 255.255.128.0   | 32,766              | 32,768        |                             |
| /16           | 255.255.0.0     | 65,534              | 65,536        | Class B size                |
| /15           | 255.254.0.0     | 131,070             | 131,072       |                             |
| /14           | 255.252.0.0     | 262,142             | 262,144       |                             |
| /13           | 255.248.0.0     | 524,286             | 524,288       |                             |
| /12           | 255.240.0.0     | 1,048,574           | 1,048,576     | Class A block               |
| /11           | 255.224.0.0     | 2,097,150           | 2,097,152     |                             |
| /10           | 255.192.0.0     | 4,194,302           | 4,194,304     |                             |
| /9            | 255.128.0.0     | 8,388,606           | 8,388,608     |                             |
| /8            | 255.0.0.0       | 16,777,214          | 16,777,216    | Entire Class A              |

___



___

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


* 🧱 Reflects **segmentation**
* 🛡️ Integrates **IDS/IPS placement**
* 📍 Shows **firewall tiers**
* 🔁 Supports **stateful inspection logic**
* 📌 Is fully mappable to your ACL and policy table

---

```css
                        ┌──────────────────────────────┐
                        │         🌐 INTERNET          │
                        └─────────────┬────────────────┘
                                      │
                         ┌────────────▼────────────┐
                         │  🛡️ EDGE FIREWALL + NIPS │
                         └────────────┬────────────┘
                                      │
                         ┌────────────▼────────────┐
                         │         DMZ ZONE         │
                         ├──────────────────────────┤
                         │  [WEB1] Public Web Server│
                         └────────────┬─────────────┘
                                      │
                         ┌────────────▼────────────┐
                         │ 🛡️ INTERNAL FIREWALL + IDS│
                         └────────────┬────────────┘
                                      │
        ┌──────────────┬─────────────┴──────────────┬───────────────┐
        │              │                            │               │
 ┌──────▼─────┐  ┌─────▼──────┐              ┌──────▼─────┐   ┌─────▼──────┐
 │ [WEB2]     │  │  [DB01]    │              │  [DB02]    │   │  [FIL1]    │
 │ Internal   │  │ Production │              │ Warehouse  │   │ File Server│
 │ Tools      │  │ Database   │              └────────────┘   └────────────┘
 └────────────┘  └────────────┘

───────────────────────────────────────────────────────────────────────────────

                           VLAN / USER ACCESS ZONES

        ┌────────────┬────────────┬─────────────┐
        │            │            │             │
 ┌──────▼────┐ ┌─────▼────┐ ┌─────▼────┐   ┌────▼───────┐
 │  [SALES]  │ │[SUPPORT] │ │  [DEV]   │   │  [WIFI]    │
 │ 50 Users  │ │ 10 Users │ │ 10 Users │   │ Guest Only │
 │ Access to │ │ Web2     │ │ Full     │   │ Internet   │
 │ DB02, FIL1│ │ Tools    │ │ Access   │   │ (No LAN)   │
 └───────────┘ └──────────┘ └──────────┘   └────────────┘

───────────────────────────────────────────────────────────────────────────────

                              MONITORING / SIEM

                       ┌─────────────────────────────┐
                       │  🔍 Security Onion Stack     │
                       │  (Suricata, Zeek, Wazuh)     │
                       └─────────────────────────────┘

                          ⇧ Log Forwarding from:
                          - Edge Firewall
                          - Internal Firewall
                          - Hosts (via Wazuh)
                          - Network sensors

```
---

## 🧠 Visualization Mapping:

* **🛡️ IDS/IPS Placement**:

  * NIPS at **edge firewall**
  * IDS (via Suricata/Zeek) at **internal firewall**

* **🎯 Zero Trust Enforcement**:

  * **No implicit trust** between VLANs or internal zones
  * **Policy-driven access** only

* **📈 Monitoring**:

  * All traffic logs shipped to **Security Onion stack**
  * Integrated with tools like:

    * **Wazuh** for HIDS
    * **Grafana/Kibana** for dashboards
    * **CyberChef/Strelka** for investigation

---

## 🔗 Tool Integration References

| Tool           | Functionality                           | Link                                                |
| -------------- | --------------------------------------- | --------------------------------------------------- |
| Suricata       | NIDS/NIPS                               | [suricata.io](https://suricata.io)                  |
| Zeek           | Network traffic analyzer                | [zeek.org](https://zeek.org)                        |
| Wazuh          | HIDS and log aggregation                | [wazuh.com](https://wazuh.com)                      |
| Strelka        | File scanning engine                    | [Strelka GitHub](https://github.com/target/strelka) |
| CyberChef      | Binary/hex/anomaly parsing              | [CyberChef](https://gchq.github.io/CyberChef/)      |
| Security Onion | Full stack security monitoring platform | [securityonion.net](https://securityonion.net)      |
| Grafana        | Metrics and dashboarding                | [grafana.com](https://grafana.com)                  |
| Kibana         | Log analytics                           | [elastic.co/kibana](https://www.elastic.co/kibana)  |

---

## 🚦Operational Takeaways

* ✅ **Dual-layer inspection**: External and internal traffic analyzed by separate sensors.
* ✅ **Segmentation-aware ACL enforcement**: VLANs cannot communicate laterally without explicit rules.
* ✅ **Stateful validation**: TCP ACK matching for reverse flow approvals.
* ✅ **Guest VLAN isolation**: Only internet access via controlled ports (80/443).
* ✅ **Audit-ready topology**: Mappable back to ACL, flow logs, and SIEM alerts.



___
> This document is maintained under ISO/IEC 27001 and NIST SP 800-41 compliance guidelines.

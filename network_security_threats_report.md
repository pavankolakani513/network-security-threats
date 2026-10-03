# Research Report: Common Network Security Threats

## 1. Introduction
In modern enterprise environments, network perimeters have expanded across distributed clouds, hybrid infrastructures, and remote endpoints. Network-layer attacks exploit fundamental design assumptions in legacy protocol suites—such as implicit trust in packet routing, unauthenticated resolution services, and open broadcast domains. Securing the transport, internet, and application layers against disruptive and interceptive threats is essential to maintain data confidentiality, transaction integrity, and system availability.

---

## 2. Denial of Service (DoS) and Distributed Denial of Service (DDoS)

### How the Attack Works
DoS and DDoS attacks aim to make an online service, network resource, or server unavailable to legitimate users by overwhelming it with malicious traffic or exploiting protocol flaws. While DoS originates from a single source, a DDoS attack coordinates thousands of distributed, compromised endpoints (botnets) using command-and-control (C2) infrastructure. Attacks span three main categories:
1. **Volumetric Attacks:** Saturate bandwidth using UDP/ICMP floods or NTP/DNS reflection amplification.
2. **Protocol/State-Exhaustion Attacks:** Consume connection state tables on firewalls, load balancers, or web servers (e.g., TCP SYN floods).
3. **Application-Layer (Layer 7) Attacks:** Target specific resource-intensive application endpoints (e.g., HTTP GET/POST floods).

### Real-World Incident
- **Event:** The 2016 Dyn DNS DDoS Attack.
- **Incident Summary:** Attackers leveraged the Mirai botnet—composed of hundreds of thousands of compromised IoT devices running default factory credentials—to launch an unprecedented multi-vector volumetric and DNS query flood against Dyn, a major Managed DNS provider.
- **Impact:** Knocked major services including GitHub, Twitter, Spotify, Netflix, and Amazon offline across the US and Europe for several hours, causing substantial financial and operational losses.

### Mitigation Strategies
1. **DDoS Scrubbing & Anycast Routing:** Route network traffic through global Anycast networks (such as Cloudflare or AWS Shield) that distribute and absorb massive volumetric traffic before it hits origin servers.
2. **Rate Limiting & Connection Throttling:** Enforce strict request thresholds and SYN cookies at reverse proxies and border firewalls to defeat TCP state-table starvation.
3. **Ingress Filtering & Access Control Lists (ACLs):** Filter malformed protocols, block unused UDP ports at perimeter routers, and drop non-essential ICMP requests.

---

## 3. Man-in-the-Middle (MITM) Attacks

### How the Attack Works
In a Man-in-the-Middle attack, an adversary positions themselves transparently between two communicating endpoints to intercept, inspect, or modify traffic in transit. Within local networks (LANs), attackers commonly execute **ARP Cache Poisoning**, broadcasting forged ARP replies that associate their MAC address with the default gateway's IP address. At higher layers, attackers leverage Rogue Wi-Fi Access Points, DNS spoofing, or SSL/TLS stripping (downgrading HTTPS connections to plaintext HTTP) to capture credentials and session tokens.

### Real-World Incident
- **Event:** The 2011 DigiNotar Certificate Authority Compromise.
- **Incident Summary:** Threat actors breached the Dutch Certificate Authority DigiNotar and issued hundreds of fraudulent wildcard SSL/TLS certificates, including one for `*.google.com`.
- **Impact:** Attackers used these fraudulent certificates to execute state-level MITM attacks against approximately 300,000 users in Iran, intercepting Gmail sessions and personal communications without browser certificate warning triggers.

### Mitigation Strategies
1. **End-to-End Encryption with HSTS:** Enforce TLS 1.3 across all communication channels and configure HTTP Strict Transport Security (HSTS) with preloading to prevent SSL stripping.
2. **Dynamic ARP Inspection (DAI) & DHCP Snooping:** Implement DAI and DHCP snooping on enterprise network switches to validate ARP packets against a trusted binding database and drop unauthorized responses.
3. **Mutual TLS (mTLS) & Certificate Pinning:** Enforce bi-directional certificate authentication for critical API and server-to-server endpoints to ensure client and server identities are cryptographically verified.

---

## 4. IP Address Spoofing

### How the Attack Works
IP address spoofing involves generating Internet Protocol (IP) packets with a forged source IP address to conceal the sender's identity, impersonate a trusted computing device, or redirect response traffic. Because standard IPv4 packet headers lack intrinsic source-origin authentication, routers evaluate routing decisions based on destination IP alone. Attackers exploit this behavior in **Reflected Amplification Attacks**: they send queries to open public resolvers (such as DNS, NTP, or SNMP) using the victim’s IP address as the forged source, causing all oversized responses to flood the victim's infrastructure.

### Real-World Incident
- **Event:** The 2018 GitHub 1.35 Tbps Memcached DDoS Attack.
- **Incident Summary:** Attackers spoofed GitHub's IP addresses and sent UDP requests to thousands of publicly exposed, misconfigured Memcached servers. The Memcached servers responded with amplification factors exceeding 50,000x directed at GitHub.
- **Impact:** Sustained 1.35 Tbps of traffic hitting GitHub's border infrastructure, temporarily forcing the service offline before traffic was routed through mitigation scrubbing centers.

### Mitigation Strategies
1. **Network Ingress and Egress Filtering (BCP 38 / RFC 2827):** Border routers must drop outbound packets containing source IP addresses outside their assigned subnet and drop inbound packets claiming to come from internal networks.
2. **Cryptographic Authentication Protocols:** Use protocols that authenticate the sender (e.g., IPsec AH/ESP, TLS, SSH) rather than relying on source IP addresses for access control decisions.
3. **Unicast Reverse Path Forwarding (uRPF):** Configure network switches and routers to discard any packet whose source IP does not match a valid return route in the routing table.

---

## 5. DNS Poisoning / Spoofing

### How the Attack Works
DNS cache poisoning occurs when invalid or malicious address records are introduced into the cache of a recursive DNS resolver. When a client requests the IP for a domain, the poisoned resolver returns the attacker's server address instead of the legitimate service IP. This redirects users to malicious clones designed to harvest credentials or distribute malware, while the address bar continues to display the expected domain name.

### Real-World Incident
- **Event:** The 2022 MyEtherWallet (MEW) / Amazon Route 53 BGP Hijack and DNS Redirection.
- **Incident Summary:** Attackers hijacked BGP routes belonging to Amazon's Route 53 DNS service and redirected queries for MyEtherWallet to malicious DNS servers hosting a phishing portal.
- **Impact:** Users navigating to the official URL were served forged DNS responses, leading to the theft of hundreds of thousands of dollars in cryptocurrency.

### Mitigation Strategies
1. **Deploy DNSSEC (DNS Security Extensions):** Sign DNS records cryptographically with public key infrastructure to ensure resolver lookups cannot be forged or tampered with.
2. **Enforce Encrypted Transport (DoH / DoT):** Adopt DNS over HTTPS (DoH) or DNS over TLS (DoT) to prevent local network actors from observing or modifying DNS queries in transit.
3. **Resolver Source Port and Transaction ID Randomization:** Prevent Kaminsky-style off-path spoofing by enforcing full 16-bit randomization for UDP source ports and query transaction IDs.

---

## 6. Threat Comparison Matrix

| Threat Name | Primary Attack Vector | Who Is at Risk | Execution Difficulty | Ease of Mitigation |
| :--- | :--- | :--- | :--- | :--- |
| **DoS / DDoS** | Volumetric bandwidth saturation; protocol/state exhaustion | Public-facing services, web applications, critical infrastructure | Low to Medium | Medium (Requires third-party scrubbing / CDN) |
| **Man-in-the-Middle** | ARP poisoning, rogue gateways, TLS downgrades | Local LAN users, public Wi-Fi clients, unencrypted services | Medium | High (Easily prevented with universal TLS & DAI) |
| **IP Spoofing** | Forged IPv4 headers, UDP amplification | Stateless UDP services, victim IPs of amplification | Low (Creation) / High (Blind Hijack) | High (Preventable via BCP 38 / uRPF filtering) |
| **DNS Poisoning** | Resolver cache tampering, forged DNS transaction IDs | End users relying on shared/caching recursive resolvers | High | Medium (Requires systemic DNSSEC deployment) |

---

## 7. Key Takeaways for Network Administrators
1. **Implement Defense in Depth:** Perimeter firewalls alone cannot block lateral or identity-based network threats. Zero-trust principles, segmentation, and continuous monitoring must be implemented across all OSI layers.
2. **Encrypt All Traffic by Default:** Plaintext protocols (HTTP, Telnet, FTP, unencrypted DNS) expose networks to interception and manipulation. Enforce TLS 1.3, mTLS, SSHv2, and DNSSEC across all production environments.
3. **Enforce Strict Boundary Ingress/Egress Controls:** Adopt RFC 2827/BCP 38 filtering and automated rate-limiting to prevent your own infrastructure from participating in or succumbing to spoofed volumetric attacks.

---

## 8. References
1. **NIST SP 800-44 Rev. 2:** Guidelines on Securing Public Web Servers. National Institute of Standards and Technology.
2. **CISA Alert (TA16-288A):** Heightened DDoS Threat Posed by Mirai and Other Botnets. Cybersecurity and Infrastructure Security Agency.
3. **IETF RFC 2827 / BCP 38:** Network Ingress Filtering: Defeating Denial of Service Attacks which employ IP Source Address Spoofing. Internet Engineering Task Force.
4. **IETF RFC 4033:** DNS Security Introduction and Requirements (DNSSEC). Internet Engineering Task Force.
5. **MITRE ATT&CK Matrix:** Technique T1498 (Network Denial of Service) and Technique T1557 (Adversary-in-the-Middle).
6.

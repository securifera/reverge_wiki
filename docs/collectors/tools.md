# Collector Tools

The reverge collector ships with a set of integrated scanning tools called **Waluigi tools**. Each tool defines:

- **Collector type** — `ACTIVE` (direct network probing) or `PASSIVE` (API/third-party lookups)
- **Input records** — the data model types the tool consumes as targets
- **Output records** — the data model types the tool produces
- **Scan order** — relative execution priority (lower numbers run first)

Tools can be enabled or disabled per scan schedule. Wordlists can be attached to tools that support directory brute-forcing.

---

## Execution Order

| Order | Tool | Type | Summary |
|---|---|---|---|
| 1 | [Subfinder](#subfinder) | PASSIVE | Subdomain enumeration via passive sources |
| 1 | [Gau](#gau) | PASSIVE | Historical URL collection from public archives |
| 2 | [Masscan](#masscan) | ACTIVE | High-speed port scanning |
| 3 | [Shodan](#shodan) | PASSIVE | Shodan API host and service lookup |
| 4 | [Httpx](#httpx) | ACTIVE | HTTP service detection and fingerprinting |
| 5 | [IP THC](#ip-thc) | PASSIVE | Passive DNS resolution via IP THC API |
| 6 | [Nmap](#nmap) | ACTIVE | Service version detection and NSE scripting |
| 6 | [Netexec](#netexec) | ACTIVE | Protocol-level authenticated enumeration |
| 6 | [Metasploit](#metasploit) | ACTIVE | MSF auxiliary/exploit module execution |
| 7 | [Nuclei](#nuclei) | ACTIVE | YAML-template-based vulnerability scanning |
| 7 | [Python](#python) | ACTIVE | Custom Python script execution |
| 8 | [Pyshot](#pyshot) | ACTIVE | Web screenshot capture (PhantomJS) |
| 8 | [Webcap](#webcap) | ACTIVE | Web screenshot capture (Chrome/Chromium) |
| 8 | [IIS Shortname Scanner](#iis-shortname-scanner) | ACTIVE | IIS 8.3 filename disclosure detection |
| 10 | [Feroxbuster](#feroxbuster) | ACTIVE | Web directory brute-forcing |
| 10 | [Crapsecrets](#crapsecrets) | ACTIVE | Cryptographic secret weakness detection |
| 12 | [Sqlmap](#sqlmap) | ACTIVE | SQL injection vulnerability detection |

---

## Passive Tools

### Subfinder

| Attribute | Value |
|---|---|
| **Type** | PASSIVE |
| **Scan Order** | 1 |
| **Project** | [github.com/projectdiscovery/subfinder](https://github.com/projectdiscovery/subfinder) |
| **Tags** | `passive`, `dns-enum` |
| **Default Args** | `-all` |
| **Input Records** | `Domain` |
| **Output Records** | `Host`, `Domain` |

Performs subdomain discovery by querying passive data sources including certificate transparency logs, public search engines, and security vendor APIs (Shodan, SecurityTrails, Chaos, etc.) without directly probing target networks.

Discovered subdomains are DNS-resolved to IP addresses, producing `Host` and `Domain` records. The `-all` flag enables all configured data sources. API keys for third-party sources are managed in a config file on the collector.

---

### Gau

| Attribute | Value |
|---|---|
| **Type** | PASSIVE |
| **Scan Order** | 1 |
| **Project** | [github.com/lc/gau](https://github.com/lc/gau) |
| **Tags** | `passive`, `http-crawl` |
| **Default Args** | `--retries 3 --timeout 5` |
| **Input Records** | `Domain`, `Port`, `HttpEndpoint` |
| **Output Records** | `Port`, `Host`, `Domain`, `ListItem`, `HttpEndpoint`, `HttpEndpointData` |

Fetches publicly known URLs for target domains from AlienVault OTX, the Wayback Machine, Common Crawl, and URLScan. No active probing of targets occurs — all data is retrieved from external archives and threat intelligence platforms.

Historical URLs are used to enrich the scan scope with previously observed HTTP endpoints, paths, and domain variants.

---

### Shodan

| Attribute | Value |
|---|---|
| **Type** | PASSIVE |
| **Scan Order** | 3 |
| **Project** | [shodan.io](https://www.shodan.io/) |
| **Tags** | `passive`, `port-scan`, `service-detection` |
| **Default Args** | *(none)* |
| **Input Records** | `Host`, `Subnet` |
| **Output Records** | `HttpEndpointData`, `HttpEndpoint`, `ListItem`, `Domain`, `Certificate`, `WebComponent`, `Port`, `Host` |

Queries the Shodan API for known information about target hosts and subnets without directly scanning them. Extracts open ports, service banners, SSL certificates, technology fingerprints, and associated domain names from Shodan's existing database.

Requires a Shodan API key configured on the collector. This is one of the richest passive data producers, returning a comprehensive set of output record types.

---

### IP THC

| Attribute | Value |
|---|---|
| **Type** | PASSIVE |
| **Scan Order** | 5 |
| **Project** | [ip.thc.org](https://ip.thc.org/) |
| **Tags** | `passive`, `dns-enum` |
| **Default Args** | *(none)* |
| **Input Records** | `Host`, `Subnet`, `Domain` |
| **Output Records** | `Domain` |

Queries the IP THC threat intelligence platform for passive DNS data associated with target IP addresses and subnets. Expands domain coverage by discovering domain names historically associated with target IPs using DNS history records.

Requires an IP THC API key. No active network probing is performed.

---

## Active Tools

### Masscan

| Attribute | Value |
|---|---|
| **Type** | ACTIVE |
| **Scan Order** | 2 |
| **Project** | [github.com/robertdavidgraham/masscan](https://github.com/robertdavidgraham/masscan) |
| **Tags** | `port-scan`, `fast` |
| **Default Args** | `--rate 1000` |
| **Input Records** | `Subnet`, `Host` |
| **Output Records** | `Host`, `Port` |

High-speed asynchronous port scanner capable of scanning large IP ranges rapidly. Uses its own TCP/IP stack to achieve packet rates of millions per second. Configured at 1,000 packets/second by default for rate-controlled scanning.

Masscan is typically the first active tool to run, rapidly identifying open ports across the target scope before more detailed tools (Nmap, Httpx) perform service identification. XML output is parsed to extract `Host` and `Port` records.

The port scope is driven by the scan's port bitmap — a bitmask encoding the set of ports to scan, transmitted from the reverge server with each scheduled scan.

---

### Httpx

| Attribute | Value |
|---|---|
| **Type** | ACTIVE |
| **Scan Order** | 4 |
| **Project** | [github.com/projectdiscovery/httpx](https://github.com/projectdiscovery/httpx) |
| **Tags** | `http-crawl`, `service-detection`, `fast` |
| **Default Args** | `-favicon -td -t 50 -timeout 3 -maxhr 5 -rstr 10000 -tls-grab` |
| **Input Records** | `Host`, `Port`, `Domain`, `HttpEndpointData` |
| **Output Records** | `HttpEndpointData`, `HttpEndpoint`, `CollectionModule`, `CollectionModuleOutput`, `WebComponent`, `Domain`, `Certificate`, `Screenshot`, `ListItem`, `Port`, `Host` |

Multi-purpose HTTP toolkit that probes all HTTP/HTTPS services in the scan scope with 50 parallel threads. This is one of the most comprehensive data-producing tools in the pipeline.

Key capabilities:

- Technology detection via Wappalyzer signatures (`-td`)
- TLS certificate extraction and SAN domain discovery (`-tls-grab`)
- Favicon hash capture for service fingerprinting (`-favicon`)
- Response metadata (title, status, content-length)
- Screenshot capture
- Timeout and response size limits to prevent stalling on large responses

Configured with a 3-second timeout, maximum 5-hop redirect follow (`-maxhr 5`), and response body truncated to 10 KB (`-rstr 10000`).

---

### Nmap

| Attribute | Value |
|---|---|
| **Type** | ACTIVE |
| **Scan Order** | 6 |
| **Project** | [github.com/nmap/nmap](https://github.com/nmap/nmap) |
| **Tags** | `port-scan`, `service-detection`, `os-detection`, `slow` |
| **Default Args** | `-sT -sV --script +ssl-cert --script-args ssl=True` |
| **Input Records** | `Subnet`, `Host`, `Port` |
| **Output Records** | `CollectionModule`, `CollectionModuleOutput`, `WebComponent`, `Domain`, `Certificate`, `ListItem`, `Port`, `Host` |

Industry-standard network scanner. Performs TCP connect scans (`-sT`) with service version detection (`-sV`) and SSL certificate retrieval via NSE scripts. Targets discovered by earlier tools (e.g., Masscan open ports) to perform deeper service identification.

Nmap supports NSE (Nmap Scripting Engine) modules. When modules are configured on a scan, Nmap enumrates available NSE scripts and runs them against matching services. Module output is stored as `CollectionModule` and `CollectionModuleOutput` records. SSL certificate data is extracted to produce `Certificate` and `Domain` records.

Tagged as `slow` due to the -sV service detection probes.

---

### Netexec

| Attribute | Value |
|---|---|
| **Type** | ACTIVE |
| **Scan Order** | 6 |
| **Project** | [github.com/Pennyw0rth/NetExec](https://github.com/Pennyw0rth/NetExec) |
| **Tags** | `service-detection`, `authenticated`, `vuln-scan`, `exploitation`, `os-detection` |
| **Default Args** | *(module-driven)* |
| **Input Records** | `Subnet`, `Host`, `Port` |
| **Output Records** | `CollectionModule`, `CollectionModuleOutput`, `WebComponent`, `Domain`, `Certificate`, `ListItem`, `Port`, `Host` |

Post-exploitation and enumeration framework supporting multiple network protocols. Netexec is used for protocol-level authenticated enumeration and vulnerability detection.

Supported protocols and their standard ports:

| Protocol | Default Port |
|---|---|
| FTP | 21 |
| SSH | 22 |
| NFS | 111 |
| WMI | 135 |
| LDAP | 389 |
| SMB | 445 |
| MySQL | 3306 |
| RDP | 3389 |
| VNC | 5900 |
| WinRM | 5985 |

Modules are dynamically enumerated from the installed binary and cached by version fingerprint. When credentials are available (via `Credential` records), Netexec performs authenticated enumeration. Module output is stored as `CollectionModule` / `CollectionModuleOutput` records, with hostnames, domains, and component data also extracted.

---

### Metasploit

| Attribute | Value |
|---|---|
| **Type** | ACTIVE |
| **Scan Order** | 6 |
| **Project** | [github.com/rapid7/metasploit-framework](https://github.com/rapid7/metasploit-framework) |
| **Tags** | `vuln-scan`, `authenticated`, `service-detection`, `exploitation` |
| **Default Args** | *(module path specified per scan, e.g., `auxiliary/scanner/smb/smb_ms17_010`)* |
| **Input Records** | `Subnet`, `Host`, `Port` |
| **Output Records** | `CollectionModule`, `CollectionModuleOutput`, `Domain`, `Port`, `Host` |

Connects to a running `msfrpcd` JSON-RPC daemon (default `127.0.0.1:8081`) to execute Metasploit auxiliary scanner and exploit modules against target hosts and ports.

Execution flow:

1. A dedicated MSF console is created via the RPC API.
2. `RHOSTS` is set from the target IP list; `RPORT` is set from the discovered port.
3. The specified module is loaded and executed (`run` for auxiliary modules, `check` for exploit modules).
4. Console output is polled until the module completes.
5. Results are stored as `CollectionModuleOutput` records.

Available modules are enumerated from the RPC server and cached by Metasploit version fingerprint. The module path is specified in the tool's `args` field at scan schedule time.

---

### Nuclei

| Attribute | Value |
|---|---|
| **Type** | ACTIVE |
| **Scan Order** | 7 |
| **Project** | [github.com/projectdiscovery/nuclei](https://github.com/projectdiscovery/nuclei) |
| **Tags** | `vuln-scan`, `slow` |
| **Default Args** | `-ni -pt http -rl 50 -t http/technologies/fingerprinthub-web-fingerprints.yaml` |
| **Input Records** | `Port`, `HttpEndpointData` |
| **Output Records** | `CollectionModule`, `CollectionModuleOutput`, `WebComponent`, `Vulnerability` |

YAML-template-based vulnerability scanner. Scans base URLs (root paths only — sub-paths are filtered to avoid redundant template runs) at rate limit 50 requests/second with HTTP-only probing (`-pt http`).

The default configuration runs the FingerprintHub technology fingerprinting templates. Custom templates can be specified in the tool `args` field. Nuclei modules (template paths) are enumerable via `-tl`/`-td` flags and can be selected per scan.

Vulnerability findings are stored as `Vuln` records; detected technologies become `WebComponent` records.

---

### Python

| Attribute | Value |
|---|---|
| **Type** | ACTIVE |
| **Scan Order** | 7 |
| **Project** | [python.org](https://www.python.org/) |
| **Tags** | `code-exec` |
| **Default Args** | *(user-supplied script path and arguments)* |
| **Input Records** | `Port` |
| **Output Records** | `CollectionModule`, `CollectionModuleOutput` |

A generic execution harness for running custom Python scripts against discovered network ports. The script path and runtime arguments are specified via the tool's `args` field at scan schedule time.

This tool enables ad-hoc automation tasks, custom protocol checks, or any custom scanning logic where a purpose-built tool module does not yet exist. Output is captured and stored as `CollectionModuleOutput` records.

---

### Pyshot

| Attribute | Value |
|---|---|
| **Type** | ACTIVE |
| **Scan Order** | 8 |
| **Project** | [github.com/securifera/pyshot](https://github.com/securifera/pyshot) |
| **Tags** | `screenshot`, `load-balancer-incompatible` |
| **Default Args** | *(none)* |
| **Input Records** | `Port`, `HttpEndpointData` |
| **Output Records** | `Screenshot`, `Domain`, `ListItem`, `HttpEndpoint`, `HttpEndpointData` |

PhantomJS-based web screenshot tool. Captures screenshots of web services concurrently and stores the results as base64-encoded `Screenshot` records. Screenshots are deduplicated by image hash.

**Note:** Tagged as `load-balancer-incompatible` — sticky sessions may be required for accurate captures on load-balanced targets. For modern environments, [Webcap](#webcap) is preferred.

---

### Webcap

| Attribute | Value |
|---|---|
| **Type** | ACTIVE |
| **Scan Order** | 8 |
| **Project** | [github.com/blacklanternsecurity/webcap](https://github.com/blacklanternsecurity/webcap) |
| **Tags** | `screenshot`, `load-balancer-compatible` |
| **Default Args** | `--timeout 5 --threads 5 --quality 20 --format jpeg` |
| **Input Records** | `Port`, `HttpEndpointData` |
| **Output Records** | `Screenshot`, `Domain`, `ListItem`, `HttpEndpoint`, `HttpEndpointData` |

Chrome/Chromium-based modern replacement for Pyshot. Captures JPEG screenshots at 20% quality using async/asyncio processing with 5 concurrent threads and a 5-second per-page timeout. Tagged as `load-balancer-compatible`.

Screenshots are base64-encoded and deduplicated by image hash before being stored as `Screenshot` records.

---

### IIS Shortname Scanner

| Attribute | Value |
|---|---|
| **Type** | ACTIVE |
| **Scan Order** | 8 |
| **Project** | [github.com/lijiejie/IIS_shortname_Scanner](https://github.com/lijiejie/IIS_shortname_Scanner) |
| **Tags** | `vuln-scan`, `http-crawl` |
| **Default Args** | *(none)* |
| **Input Records** | `Port`, `HttpEndpointData` |
| **Output Records** | `CollectionModule`, `CollectionModuleOutput` |

Probes IIS web servers for the 8.3 filename disclosure vulnerability, which allows unauthenticated attackers to enumerate short (8.3 format) filenames and directory names via HTTP requests.

The scanner first performs a vulnerability check (`is_vulnerable()`) before running the full enumeration. Results include lists of discovered short filenames (`files`) and directory names (`dirs`), stored as `CollectionModuleOutput` records.

---

### Feroxbuster

| Attribute | Value |
|---|---|
| **Type** | ACTIVE |
| **Scan Order** | 10 |
| **Project** | [github.com/epi052/feroxbuster](https://github.com/epi052/feroxbuster) |
| **Tags** | `http-crawl` |
| **Default Args** | `--rate-limit 50 -s 200 -n --auto-bail` |
| **Input Records** | `Port`, `HttpEndpointData` |
| **Output Records** | `Domain`, `ListItem`, `HttpEndpoint`, `HttpEndpointData` |

Rust-based web directory scanner. Performs dictionary-based path brute-forcing against HTTP/HTTPS services. Rate-limited to 50 requests/second, filtering for HTTP 200 responses only (`-s 200`), with no recursion (`-n`) and automatic error bail-out (`--auto-bail`).

URLs across all targets are deduplicated to avoid rescanning previously discovered paths. Discovered paths are stored as `ListItem` and `HttpEndpoint` records.

A wordlist must be configured for this tool to be effective. Wordlists are managed in the reverge [Wordlists](/resources/wordlists/) settings.

---

### Crapsecrets

| Attribute | Value |
|---|---|
| **Type** | ACTIVE |
| **Scan Order** | 10 |
| **Project** | [github.com/irsdl/crapsecrets](https://github.com/irsdl/crapsecrets) |
| **Tags** | `vuln-scan` |
| **Default Args** | `-nh -t 3 -mrd 5 -avsk -fvsp` |
| **Input Records** | `Port`, `HttpEndpointData` |
| **Output Records** | `Vulnerability` |

Detects the use of known-weak or default cryptographic secrets in web applications — including JWT signing keys, framework session keys, and API tokens. Scans base URLs only (root paths), filtering out sub-paths to focus on the primary application entry points.

Default flag meanings:

| Flag | Meaning |
|---|---|
| `-nh` | No hashcat (skip hashcat-based analysis) |
| `-t 3` | Use 3 threads |
| `-mrd 5` | Maximum request delay of 5 seconds |
| `-avsk` | Check all very-short known secrets |
| `-fvsp` | Filter valid secrets only |

Findings are stored as `Vulnerability` records.

---

### Sqlmap

| Attribute | Value |
|---|---|
| **Type** | ACTIVE |
| **Scan Order** | 12 |
| **Project** | [sqlmap.org](https://sqlmap.org/) |
| **Tags** | `vuln-scan` |
| **Default Args** | `--batch --level=1 --risk=1 --crawl=2` |
| **Input Records** | `Port`, `HttpEndpointData` |
| **Output Records** | `Vulnerability` |

Automated SQL injection detection and exploitation tool. Runs non-interactively (`--batch`) at conservative level 1 / risk 1 with a depth-2 crawl to discover injectable parameters across web endpoints.

Positioned at scan order 12 to run after Feroxbuster (10) so that all discovered HTTP endpoints are available as injection targets. URL deduplication is tracked globally to avoid retesting previously scanned endpoints.

Confirmed SQL injection findings are stored as `Vulnerability` records.

---

## Tool Configuration

Each tool's behavior can be customized at scan schedule time by modifying the `args` field in the scan configuration. Default argument values are defined per tool and can be overridden without modifying the collector installation.

For tools that support **modules** (Nmap NSE scripts, Netexec protocol modules, Nuclei templates, Metasploit auxiliary modules), modules can be selected individually per scan to control what is executed against discovered services.

See [Scans](/scans/scans/) for instructions on configuring tools as part of a scan schedule.

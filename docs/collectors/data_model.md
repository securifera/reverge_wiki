# Collector Data Model

The reverge collector uses a structured data model to represent all discovered network assets, vulnerabilities, and metadata. Every object produced by a scanning tool is serialized as a JSON record and sent to the reverge server for ingestion via the `/import` API endpoint.

## Record Structure

All data model objects share a common JSON envelope:

```json
{
  "id": "<hex-uuid>",
  "type": "<record-type>",
  "parent": {
    "type": "<parent-record-type>",
    "id": "<parent-hex-uuid>"
  },
  "collection_tool_instance_id": "<tool-instance-id>",
  "data": { ... }
}
```

| Field | Description |
|---|---|
| `id` | Unique hex-encoded UUID for the record |
| `type` | Lowercase class name identifying the record type (e.g., `host`, `port`, `vuln`) |
| `parent` | Optional reference to the parent record; `null` for root-level objects |
| `collection_tool_instance_id` | ID of the scheduled tool run that produced this record |
| `data` | Record-type-specific payload (see each type below) |

## Record Tags

Records are internally tagged to indicate their origin and scope during processing. Tags are **not** included in the JSON sent to the server but affect how the collector filters and deduplicates records in memory.

| Tag | Value | Description |
|---|---|---|
| `LOCAL` | `1` | Collected directly by a local scanner on the collector |
| `REMOTE` | `2` | Obtained from a remote API source (e.g., Shodan, IP THC) |
| `SCOPE` | `3` | Falls within the defined scan scope provided by the server |

## Record Types

### `host`

Represents a network host (IP address). This is the root object for most data hierarchies.

**Parent:** none

```json
{
  "id": "1a2b3c4d...",
  "type": "host",
  "parent": null,
  "collection_tool_instance_id": "abc123",
  "data": {
    "ipv4_addr": "192.168.1.100"
  }
}
```

| Data Field | Type | Required | Description |
|---|---|---|---|
| `ipv4_addr` | string | yes* | IPv4 address of the host |
| `ipv6_addr` | string | yes* | IPv6 address (mutually exclusive with `ipv4_addr`) |
| `credential` | object | no | Credential reference if authenticated access is available (`{"credential_id": "<id>"}`) |

---

### `subnet`

Represents a network subnet, typically used as a scan scope input.

**Parent:** none

```json
{
  "id": "5e6f7a8b...",
  "type": "subnet",
  "parent": null,
  "collection_tool_instance_id": "abc123",
  "data": {
    "subnet": "10.0.0.0",
    "mask": "24"
  }
}
```

| Data Field | Type | Required | Description |
|---|---|---|---|
| `subnet` | string | yes | Network address (e.g., `10.0.0.0`) |
| `mask` | string | yes | CIDR prefix length (e.g., `24`) |

---

### `port`

Represents an open network port on a host. Children of `host` records.

**Parent:** `host`

```json
{
  "id": "9c0d1e2f...",
  "type": "port",
  "parent": {
    "type": "host",
    "id": "1a2b3c4d..."
  },
  "collection_tool_instance_id": "abc123",
  "data": {
    "port": "443",
    "proto": 6,
    "secure": true
  }
}
```

| Data Field | Type | Required | Description |
|---|---|---|---|
| `port` | string | yes | Port number as a string |
| `proto` | int | yes | IP protocol number (`6` = TCP, `17` = UDP) |
| `secure` | bool | no | Whether the port uses TLS/SSL (`true` = HTTPS, SMTPS, etc.) |
| `credential_id` | string | no | Credential ID used for authenticated access to this port |

---

### `domain`

Represents a domain name resolved to or associated with a host. Children of `host` records.

**Parent:** `host`

```json
{
  "id": "3a4b5c6d...",
  "type": "domain",
  "parent": {
    "type": "host",
    "id": "1a2b3c4d..."
  },
  "collection_tool_instance_id": "abc123",
  "data": {
    "name": "www.example.com"
  }
}
```

| Data Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes | The fully-qualified domain name |
| `credential_id` | string | no | Credential ID associated with this domain |

---

### `listitem`

Represents a discovered web path or URI. Used as a deduplication key for HTTP endpoints. `ListItem` records are standalone (no parent) and are referenced by `HttpEndpoint` objects via their `web_path_id` field.

**Parent:** none

```json
{
  "id": "7e8f9a0b...",
  "type": "listitem",
  "parent": null,
  "collection_tool_instance_id": "abc123",
  "data": {
    "path": "/admin/login",
    "path_hash": "da39a3ee5e6b4b0d3255bfef95601890afd80709"
  }
}
```

| Data Field | Type | Required | Description |
|---|---|---|---|
| `path` | string | yes | Web path URI (e.g., `/admin`, `/api/v1/users`). Defaults to `/` if null |
| `path_hash` | string | yes | SHA-1 hex digest of the path string, used for deduplication |

---

### `httpendpoint`

Represents an HTTP endpoint (a specific path on a port's web service). Children of `port` records.

**Parent:** `port`

```json
{
  "id": "b1c2d3e4...",
  "type": "httpendpoint",
  "parent": {
    "type": "port",
    "id": "9c0d1e2f..."
  },
  "collection_tool_instance_id": "abc123",
  "data": {
    "web_path_id": "7e8f9a0b..."
  }
}
```

| Data Field | Type | Required | Description |
|---|---|---|---|
| `web_path_id` | string | yes | ID of the associated `ListItem` record representing the path |

---

### `httpendpointdata`

Stores HTTP response metadata for an HTTP endpoint. Multiple `httpendpointdata` records can exist per endpoint (e.g., IP-based vs. domain-based responses). Children of `httpendpoint` records.

**Parent:** `httpendpoint`

```json
{
  "id": "f5a6b7c8...",
  "type": "httpendpointdata",
  "parent": {
    "type": "httpendpoint",
    "id": "b1c2d3e4..."
  },
  "collection_tool_instance_id": "abc123",
  "data": {
    "title": "Admin Login Panel",
    "status": "200",
    "domain_id": "3a4b5c6d...",
    "screenshot_id": "d9e0f1a2...",
    "last_modified": "Thu, 01 Jan 2025 00:00:00 GMT",
    "fav_icon_hash": "-1234567890",
    "content_length": "4096"
  }
}
```

| Data Field | Type | Required | Description |
|---|---|---|---|
| `title` | string | no | HTML `<title>` of the response page |
| `status` | string | no | HTTP response status code (e.g., `"200"`, `"301"`) |
| `domain_id` | string | no | ID of the `Domain` record if this response was fetched using a domain name (rather than IP) |
| `screenshot_id` | string | no | ID of the associated `Screenshot` record |
| `last_modified` | string | no | Value of the `Last-Modified` HTTP response header |
| `fav_icon_hash` | string | no | MMH3 hash of the favicon for fingerprinting (Shodan-compatible format) |
| `content_length` | string | no | Size of the response body in bytes |

---

### `screenshot`

Stores a base64-encoded screenshot of a web page, captured by Pyshot or Webcap. Standalone records (no parent), referenced by `httpendpointdata` records via `screenshot_id`.

**Parent:** none

```json
{
  "id": "d9e0f1a2...",
  "type": "screenshot",
  "parent": null,
  "collection_tool_instance_id": "abc123",
  "data": {
    "screenshot": "<base64-encoded-image-data>",
    "image_hash": "a1b2c3d4e5f6..."
  }
}
```

| Data Field | Type | Required | Description |
|---|---|---|---|
| `screenshot` | string | no | Base64-encoded image data (JPEG or PNG) |
| `image_hash` | string | no | Hash of the image data used for deduplication |

---

### `webcomponent`

Represents a detected web technology or software component (e.g., web server, framework, CMS). Children of `port` records.

**Parent:** `port`

```json
{
  "id": "c3d4e5f6...",
  "type": "webcomponent",
  "parent": {
    "type": "port",
    "id": "9c0d1e2f..."
  },
  "collection_tool_instance_id": "abc123",
  "data": {
    "name": "nginx",
    "version": "1.18.0"
  }
}
```

| Data Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes | Name of the detected technology (e.g., `nginx`, `WordPress`, `Apache`) |
| `version` | string | no | Detected version string if available |

---

### `vuln`

Represents a security vulnerability or finding detected on a port. Children of `port` records.

**Parent:** `port`

```json
{
  "id": "e7f8a9b0...",
  "type": "vuln",
  "parent": {
    "type": "port",
    "id": "9c0d1e2f..."
  },
  "collection_tool_instance_id": "abc123",
  "data": {
    "name": "CVE-2021-44228",
    "vuln_details": "Log4Shell - JNDI injection vulnerability in Apache Log4j 2.x",
    "endpoint_id": "b1c2d3e4..."
  }
}
```

| Data Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes | Vulnerability identifier (CVE ID, template name, or descriptive label) |
| `vuln_details` | string | no | Extended description or evidence of the vulnerability |
| `endpoint_id` | string | no | ID of the `HttpEndpoint` where the vulnerability was found |

---

### `certificate`

Represents an SSL/TLS certificate discovered on a port. Children of `port` records.

**Parent:** `port`

```json
{
  "id": "1b2c3d4e...",
  "type": "certificate",
  "parent": {
    "type": "port",
    "id": "9c0d1e2f..."
  },
  "collection_tool_instance_id": "abc123",
  "data": {
    "issuer": "Let's Encrypt Authority X3",
    "issued": 1609459200,
    "expires": 1640995200,
    "fingerprint_hash": "AA:BB:CC:DD:EE:FF:00:11:22:33:44:55:66:77:88:99:AA:BB:CC:DD",
    "domain_id_list": ["3a4b5c6d...", "5e6f7a8b..."]
  }
}
```

| Data Field | Type | Required | Description |
|---|---|---|---|
| `issuer` | string | yes | Certificate issuer common name or organization |
| `issued` | int | yes | Unix epoch timestamp when the certificate was issued |
| `expires` | int | yes | Unix epoch timestamp when the certificate expires |
| `fingerprint_hash` | string | yes | Certificate fingerprint (SHA-1 or SHA-256 hex string) |
| `domain_id_list` | array | yes | List of `Domain` record IDs for all Subject Alternative Names (SANs) on the certificate |

---

### `collectionmodule`

Represents a named scanning module invocation, grouping the output from a specific tool module run. Children of a `Tool` record (`parent.type` will be the tool name). Used by tools like Nmap (NSE scripts), Netexec (protocol modules), and Metasploit (auxiliary modules).

**Parent:** `tool` (virtual parent referencing the tool instance)

```json
{
  "id": "5f6a7b8c...",
  "type": "collectionmodule",
  "parent": {
    "type": "tool",
    "id": "<tool-instance-id>"
  },
  "collection_tool_instance_id": "abc123",
  "data": {
    "name": "smb-security-mode",
    "description": "Determines the message signing configuration of an SMB server",
    "args": ""
  }
}
```

| Data Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes | Module name (e.g., NSE script name, Netexec module name) |
| `description` | string | no | Human-readable description of the module |
| `args` | string | yes | Arguments passed to the module |

---

### `collectionmoduleoutput`

Stores the raw text output produced by a `CollectionModule` for a specific port. Children of `collectionmodule` records.

**Parent:** `collectionmodule`

```json
{
  "id": "9d0e1f2a...",
  "type": "collectionmoduleoutput",
  "parent": {
    "type": "collectionmodule",
    "id": "5f6a7b8c..."
  },
  "collection_tool_instance_id": "abc123",
  "data": {
    "output": "message signing: disabled",
    "port_id": "9c0d1e2f..."
  }
}
```

| Data Field | Type | Required | Description |
|---|---|---|---|
| `output` | string | yes | Raw text output from the module for this target |
| `port_id` | string | yes | ID of the `Port` record this output is associated with |

---

### `credential`

Represents a set of authentication credentials discovered or used during scanning.

**Parent:** none

```json
{
  "id": "3b4c5d6e...",
  "type": "credential",
  "parent": null,
  "collection_tool_instance_id": "abc123",
  "data": {
    "username": "admin",
    "password": "Password123!",
    "privileged": false
  }
}
```

| Data Field | Type | Required | Description |
|---|---|---|---|
| `username` | string | yes | Account username |
| `password` | string | yes | Account password |
| `privileged` | bool | no | Whether the credential has elevated/admin privileges (defaults to `false`) |

---

## Object Hierarchy

The following diagram shows the parent-child relationships between data model objects:

```
Subnet (no parent)
Host (no parent)
├── Port
│   ├── WebComponent
│   ├── Vuln
│   ├── Certificate
│   │   └── → (references Domain records)
│   └── HttpEndpoint  ─── references → ListItem
│       └── HttpEndpointData
│           ├── → (references Domain)
│           └── → (references Screenshot)
└── Domain

ListItem (no parent)
Screenshot (no parent)
Credential (no parent)
CollectionModule (parent: Tool/virtual)
└── CollectionModuleOutput
    └── → (references Port)
```

## Data Ingestion Flow

1. A scanning tool runs and produces raw output (XML, JSON, stdout).
2. The tool's `import_func` parses the output and constructs data model objects.
3. Objects are serialized via `.to_jsonable()` and sent to the server's `/import` endpoint.
4. The server assigns database IDs and returns an `orig_id` → `db_id` mapping.
5. The collector updates all local records with the server-assigned IDs and writes the final JSON to a `tool_import_json` file.
6. The scan scope (`ScanData`) is updated with the new records so subsequent tools can use them as inputs.

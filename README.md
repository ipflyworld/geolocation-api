<div align="center">

# 🌍 IPFly.World API

**Fast, accurate IP to geolocation & threat intelligence — one HTTP GET away.**

[![Website](https://img.shields.io/badge/website-ipfly.world-2ea44f?style=for-the-badge)](https://ipfly.world)
[![Free tier](https://img.shields.io/badge/free_tier-1%2C000_req%2Fday-blue?style=for-the-badge)](https://ipfly.world)
[![Format](https://img.shields.io/badge/response-JSON-orange?style=for-the-badge)](#response-fields)
[![IPv4 + IPv6](https://img.shields.io/badge/IPv4-%2B_IPv6-8250df?style=for-the-badge)](#-ipv6)

Free Usage · No OAuth handshake · No request signing — just a token and, optionally, an IP.

[Quick Start](#-quick-start) · [Authentication](#-authentication) · [Parameters](#-parameters) · [Response Fields](#response-fields) · [Errors](#-errors) · [Best Practices](#-best-practices) · [Code Examples](#-code-examples) · [SDKs](#-official-sdks)

</div>

---

## ✨ What you get

| | |
|---|---|
| 📍 **Location** | Country, state/province, city, ZIP, latitude/longitude |
| 🕒 **Timezone** | IANA zone, UTC offsets, DST state, current local time |
| 💱 **Locale** | Currency, calling code, TLD, languages, EU membership |
| 🏢 **Network** | ASN, route (CIDR), organization, ISP/hosting/business/education type |
| 🛡️ **Security** | VPN, proxy, Tor, iCloud Relay, datacenter, attacker/abuser/threat flags, blocklists, reputation scores |

> 🔒 **Security & ASN/Company fields** are plan-dependent, starting from the **Pro Monthly** plan. See [pricing](https://ipfly.world).

---

## 🚀 Quick Start

Every lookup is a single HTTP `GET`:

```http
GET https://ipfly.world/api?token=YOUR_TOKEN&ip=8.8.8.8&include=security
```

```bash
curl "https://ipfly.world/api?token=YOUR_TOKEN&ip=8.8.8.8&include=security"
```

<details>
<summary><b>📦 Example response for <code>8.8.8.8</code></b></summary>

```json
{
  "ip": "8.8.8.8",
  "hostname": "dns.google",
  "country_code2": "US",
  "country_code3": "USA",
  "country_name": "United States",
  "country_capital": "Washington",
  "state_prov": "California",
  "city": "Mountain View",
  "zipcode": "94043",
  "latitude": "37.4056",
  "longitude": "-122.0775",
  "is_eu": false,
  "country_emoji": "🇺🇸",
  "calling_code": "+1",
  "country_tld": ".us",
  "languages": "en",
  "currency": { "code": "USD", "name": "Dollar", "symbol": "$" },
  "time_zone": {
    "name": "America/New_York",
    "offset": -4,
    "offset_with_dst": -3,
    "current_time": "2026-08-21 03:12:46.964685-0400",
    "current_time_unix": 1787296366.9647,
    "current_tz_abbreviation": "EDT",
    "current_tz_full_name": "America/New_York",
    "is_dst": true
  },
  "asn": {
    "asn": "AS15169",
    "name": "Google LLC",
    "domain": "google.com",
    "route": "8.8.8.0/24",
    "type": "hosting"
  },
  "company": { "name": "AS15169 Google LLC", "domain": "google.com", "type": "isp" },
  "security": {
    "is_tor": false,
    "is_vpn": false,
    "is_icloud_relay": false,
    "is_proxy": false,
    "is_datacenter": true,
    "is_anonymous": false,
    "is_known_attacker": false,
    "is_known_abuser": false,
    "is_threat": false,
    "is_bogon": false,
    "blocklists": [],
    "scores": { "vpn_score": 100, "proxy_score": 1, "threat_score": 100, "trust_score": 33 }
  }
}
```

</details>

> 💡 Don't have a token yet? [Create a free account](https://ipfly.world) — the free tier includes **1,000 requests/day**.

---

## 🔑 Authentication

All requests require your API token as a `token` query parameter.

```http
# Detect the caller's own IP
GET https://ipfly.world/api?token=YOUR_TOKEN

# Look up a specific IP
GET https://ipfly.world/api?token=YOUR_TOKEN&ip=IP_ADDRESS
```

> ⚠️ **Never call the API directly from client-side JavaScript, a mobile app bundle, or any code a user can inspect.** Your token will be visible in network requests and can be extracted and abused. **Always proxy requests through your own backend.**

### 🎯 Restrict your token's origin

Even proxied tokens are safer when locked down. In your dashboard, restrict a token to specific **IPs and domains** so it stops working the moment it's used from anywhere else — including a leaked or stolen copy.

- Leaving both fields **empty** allows requests from any origin.
- Once you add **at least one** IP or domain, the token works **only** from that allow-list.

---

## 📨 Parameters

All requests are plain HTTP `GET` calls. No special headers required.

| Parameter | Type | Required | Description |
|---|---|:---:|---|
| `token` | string | ✅ | Your API authentication token |
| `ip` | string | ➖ | IPv4 or IPv6 address to look up. Omit to detect the caller's IP |
| `include` | string | ➖ | Extra field sets, e.g. `security` (plan-dependent) |

### 🌐 IPv6

IPv6 works out of the box and returns the same response shape — pass addresses exactly as written (e.g. `2001:4860:4860::8888`), no encoding needed.

> If much of your traffic is IPv4-mapped via NAT64 or a CDN, the resolved location reflects the **edge or gateway**, not the end user.

---

## 🧪 Sandbox & test IPs

Well-known, stable IPs that are great for integration tests and demos:

| IP | Good for testing | Typical result |
|---|---|---|
| `8.8.8.8` | Standard US result, hosting-type ASN | 🇺🇸 Mountain View, CA — Google LLC |
| `1.1.1.1` | Anycast edge network, non-US | 🇦🇺 Sydney/Anycast — Cloudflare |
| `9.9.9.9` | Privacy-focused resolver, EU-adjacent | 🇨🇭 Quad9 Foundation |
| `2001:4860:4860::8888` | IPv6 code path | Same fields as IPv4, Google IPv6 DNS |
| `127.0.0.1` / `10.0.0.0/8` | Negative test — private/reserved ranges | Returns an error; never publicly routable |

> 🐛 **Common first bug:** testing from `localhost`. Your server's *outbound* IP (not `127.0.0.1`) is what gets geolocated. Behind a home router or corporate NAT, expect an approximate result centered on your ISP — that's normal for every IP geolocation provider.

---

## Response fields

<details open>
<summary><b>Standalone fields</b> — root-level, returned for every lookup</summary>

| Field | Type | Description |
|---|---|---|
| `ip` | string | The queried IP address |
| `hostname` | string | Reverse DNS hostname of the IP |
| `country_code2` | string | ISO 3166-1 alpha-2 country code |
| `country_code3` | string | ISO 3166-1 alpha-3 country code |
| `country_name` | string | Common name of the country |
| `country_capital` | string | Capital city of the country |
| `country_emoji` | string | Country flag emoji |
| `calling_code` | string | International dialing code |
| `country_tld` | string | Country code top-level domain (e.g. `.ua`) |
| `languages` | string | Official languages of the country |
| `is_eu` | boolean | Whether the country is an EU member |

</details>

<details>
<summary><b>📍 Location fields</b></summary>

| Field | Type | Status | Description |
|---|---|:---:|---|
| `state_prov` | string | Optional | State / province / region name |
| `state_code` | string | Optional | State/province code |
| `district` | string | Optional | District or sub-region |
| `city` | string | Optional | City name |
| `zipcode` | string | Optional | ZIP / postal code |
| `latitude` | string | Required | Latitude coordinate |
| `longitude` | string | Required | Longitude coordinate |

</details>

<details>
<summary><b>🏢 <code>asn</code> object</b></summary>

| Field | Type | Description |
|---|---|---|
| `asn.asn` | string | AS number (e.g. `AS15169`) |
| `asn.name` | string | Organization name |
| `asn.domain` | string | Organization's domain |
| `asn.route` | string | IP route/prefix (CIDR notation) |
| `asn.type` | string | Network type: `isp`, `hosting`, `business`, `education` |

</details>

<details>
<summary><b>🕒 <code>time_zone</code> object</b></summary>

| Field | Type | Description |
|---|---|---|
| `time_zone.name` | string | IANA timezone name (e.g. `Europe/Paris`) |
| `time_zone.offset` | number | UTC offset in hours (without DST) |
| `time_zone.offset_with_dst` | number | UTC offset including DST |
| `time_zone.current_time` | string | Current local time string |
| `time_zone.current_time_unix` | number | Current time as Unix timestamp |
| `time_zone.current_tz_abbreviation` | string | Timezone abbreviation (e.g. `EEST`) |
| `time_zone.is_dst` | boolean | Whether DST is currently active |

</details>

<details>
<summary><b>🛡️ <code>security</code> object</b> — threat intelligence & anonymization detection</summary>

| Field | Type | Description |
|---|---|---|
| `security.is_tor` | boolean | IP is a Tor exit node |
| `security.is_vpn` | boolean | IP is used for a VPN |
| `security.is_icloud_relay` | boolean | IP is an iCloud (Apple) proxy |
| `security.is_proxy` | boolean | IP is a known proxy |
| `security.is_datacenter` | boolean | IP belongs to a datacenter allocation |
| `security.is_anonymous` | boolean | IP is used for anonymization |
| `security.is_known_attacker` | boolean | IP is used for DoS attacks, floods, etc. |
| `security.is_known_abuser` | boolean | IP is used for abuse like hijacking, malware, etc. |
| `security.is_threat` | boolean | IP belongs to a high-risk threat |
| `security.is_bogon` | boolean | IP belongs to a bogon CIDR range |
| `security.blocklists` | array | Organic ban-lists the IP appears on |
| `security.scores` | object | Reputation scores: `vpn_score`, `proxy_score`, `threat_score`, `trust_score` |

</details>

<details>
<summary><b>💱 <code>currency</code> object</b></summary>

| Field | Type | Description |
|---|---|---|
| `currency.code` | string | ISO 4217 currency code (e.g. `EUR`) |
| `currency.name` | string | Currency name (e.g. `Euro`) |
| `currency.symbol` | string | Currency symbol (e.g. `€`) |

</details>

---

## 🚦 Errors

Standard HTTP status codes: `2xx` success, `4xx` a problem with the request, `5xx` a problem on IPFly's end. Every error also returns a JSON body.

| Status | Meaning | Typical cause |
|:---:|---|---|
| ✅ `200` | OK | Lookup succeeded |
| `400` | Bad Request | Malformed or unparseable IP address |
| `401` | Unauthorized | Missing or invalid token |
| `403` | Forbidden | Token valid but suspended, or plan doesn't include this field set |
| `404` | Not Found | No record for a private, reserved, or bogon IP |
| `500` / `503` | Server Error | Transient issue on IPFly's side — **safe to retry with backoff** |

```json
// 401 Unauthorized
{
  "error": "Invalid API token! Please recheck your token or sign up at https://ipfly.world/billing"
}
```

> 💡 Branch on the HTTP status (and any stable error code) rather than the human-readable message — message wording is meant for logs and dashboards and may evolve.

---

## 🧠 Best Practices

Patterns that separate integrations that hold up in production from ones that break under real traffic.

| | Practice | Why |
|:---:|---|---|
| 🗄️ | **Cache by IP** | A given IP's location rarely changes hour to hour. Caching responses for 12–24 hours keyed on the IP cuts request volume dramatically — most apps see 80%+ hit rates once they have returning users. |
| 🔐 | **Keep the token server-side** | Proxy lookups through your backend. A token in a browser bundle or mobile app will eventually be scraped and used to burn your quota. |
| 🚩 | **Treat security flags as signals, not verdicts** | `is_vpn`, `is_proxy`, and `is_tor` are strong risk signals, not proof of intent. Combine them with your own account/behavior signals before blocking — false positives on shared corporate NATs and mobile carrier gateways are common with every provider. |
| 🪂 | **Fail open, not closed** | If a lookup errors or times out, don't block checkout/login/page load on it. Fall back safely (no personalization, conservative currency) and log the failure. |
| 📦 | **Batch thoughtfully** | For large IP lists, use a bounded worker pool (5–10 concurrent requests) instead of firing everything at once — friendlier to the API and easier to reason about failures. |
| ⚖️ | **Mind data-retention rules** | An IP paired with derived location data can count as personal data under GDPR and similar regimes. Apply the same retention/deletion policies you use for other user PII. |

---

## 💻 Code Examples

The same lookup in eight flavors. Every snippet uses only the standard library or one common HTTP package — **no IPFly-specific SDK required**.

<details open>
<summary><b>cURL</b></summary>

```bash
curl "https://ipfly.world/api?token=YOUR_TOKEN"
curl "https://ipfly.world/api?token=YOUR_TOKEN&ip=8.8.8.8"

# Pretty-print with jq
curl -s "https://ipfly.world/api?token=YOUR_TOKEN&ip=8.8.8.8" | jq .
```

</details>

<details>
<summary><b>JavaScript</b> (browser / Deno)</summary>

```javascript
const res = await fetch('https://ipfly.world/api?token=YOUR_TOKEN&ip=8.8.8.8');
if (!res.ok) throw new Error(`IPFly error: ${res.status}`);
const data = await res.json();

console.log(data.country_name);     // "United States"
console.log(data.city);             // "Mountain View"
console.log(data.security.is_vpn);  // false
console.log(data.currency.symbol);  // "$"
```

> ⚠️ Exposes your token in the browser — see [Authentication](#-authentication).

</details>

<details>
<summary><b>Node.js</b> 18+</summary>

```javascript
// Native fetch, no dependencies needed on Node 18+
async function lookupIP(ip) {
  const url = new URL('https://ipfly.world/api');
  url.searchParams.set('token', process.env.IPFLY_TOKEN);
  if (ip) url.searchParams.set('ip', ip);

  const res = await fetch(url);
  if (!res.ok) throw new Error(`IPFly ${res.status}`);
  return res.json();
}

const geo = await lookupIP('8.8.8.8');
console.log(geo.country_name, geo.asn.name);
```

</details>

<details>
<summary><b>PHP</b></summary>

```php
<?php
$url  = 'https://ipfly.world/api?token=' . urlencode($_ENV['IPFLY_TOKEN']) . '&ip=8.8.8.8';
$ctx  = stream_context_create(['http' => ['timeout' => 5]]);
$json = @file_get_contents($url, false, $ctx);

if ($json === false) {
    // network-level failure — fail open
    exit;
}

$data = json_decode($json, true);
if (isset($data['error'])) {
    error_log('IPFly error: ' . json_encode($data['error']));
    exit;
}

echo $data['country_name'];        // "United States"
echo $data['security']['is_vpn'];  // false
```

</details>

<details>
<summary><b>Python</b></summary>

```python
import os
import requests

def lookup_ip(ip: str | None = None, timeout: float = 5.0) -> dict:
    params = {"token": os.environ["IPFLY_TOKEN"]}
    if ip:
        params["ip"] = ip
    r = requests.get("https://ipfly.world/api", params=params, timeout=timeout)
    r.raise_for_status()
    return r.json()

data = lookup_ip("8.8.8.8")
print(data["city"], data["asn"]["name"])  # Mountain View Google LLC
```

</details>

<details>
<summary><b>Java</b> 11+</summary>

```java
import java.net.URI;
import java.net.http.*;
import java.time.Duration;

var client = HttpClient.newBuilder()
    .connectTimeout(Duration.ofSeconds(5))
    .build();

var request = HttpRequest.newBuilder()
    .uri(URI.create("https://ipfly.world/api?token=" + System.getenv("IPFLY_TOKEN") + "&ip=8.8.8.8"))
    .timeout(Duration.ofSeconds(5))
    .build();

var response = client.send(request, HttpResponse.BodyHandlers.ofString());
if (response.statusCode() != 200) {
    throw new RuntimeException("IPFly error: " + response.statusCode());
}
System.out.println(response.body());
```

</details>

<details>
<summary><b>Ruby</b></summary>

```ruby
require 'net/http'
require 'json'

uri = URI("https://ipfly.world/api")
uri.query = URI.encode_www_form(token: ENV['IPFLY_TOKEN'], ip: '8.8.8.8')

response = Net::HTTP.get_response(uri)
data     = JSON.parse(response.body)
raise "IPFly error: #{data['error']}" if data['error']

puts data['country_name']
puts data['security']['is_vpn']
```

</details>

<details>
<summary><b>Go</b></summary>

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
    "net/url"
    "os"
    "time"
)

func main() {
    q := url.Values{}
    q.Set("token", os.Getenv("IPFLY_TOKEN"))
    q.Set("ip", "8.8.8.8")

    client := &http.Client{Timeout: 5 * time.Second}
    resp, err := client.Get("https://ipfly.world/api?" + q.Encode())
    if err != nil {
        panic(err)
    }
    defer resp.Body.Close()

    var data map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&data)
    fmt.Println(data["country_name"])
}
```

</details>

---

## 📚 Official SDKs

Want caching, retries, rate limiting, and batch lookups without writing them yourself? Drop-in, dependency-free clients:

| SDK | File | Docs |
|---|---|---|
| 🟨 **JavaScript** | [`ipfly-sdk.js`](https://ipfly.world/sdk/javascript) |
| 🐍 **Python** | [`ipfly_sdk.py`](https://ipfly.world/sdk/python) |

---

## 🤖 For AI & crawlers

Machine-readable references live at [`/llms.txt`](https://ipfly.world/llms.txt) (concise index) and [`/llms-full.txt`](https://ipfly.world/llms-full.txt) (complete reference).

---

<div align="center">

**[Get your free API token →](https://ipfly.world)**

Built for developers who'd rather ship features than wrangle geolocation data.

</div>

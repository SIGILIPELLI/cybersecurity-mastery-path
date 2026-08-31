# 06 · SIEM & Log Analysis Basics

Every meaningful security event leaves a trail in logs — a failed login, a
new process, an outbound connection to an unusual host. A **SIEM**
(Security Information and Event Management) system centralizes those logs
from every source, correlates them, and alerts on patterns a human
couldn't spot by reading logs one at a time. This module builds a small
local SIEM stack and writes real detection queries.

## 1. Why centralize logs at all

```
Without a SIEM: logs live on 200 different servers, in 200 different
  formats, and get overwritten/rotated within days. An attacker who
  compromises one host and clears its logs erases the evidence entirely.

With a SIEM: every log ships off-host, in near-real-time, to a central
  store the attacker (usually) can't reach or modify -- and can be
  correlated against logs from every other system.
```

This single property — logs surviving even if the source host is fully
compromised — is why "ship logs off-box" is one of the highest-leverage
detection investments an organization can make.

## 2. Building a local SIEM stack (ELK)

The **ELK stack** (Elasticsearch, Logstash, Kibana) is the most common
free/open-source SIEM foundation.

```bash
# docker-compose.yml (single-node lab setup)
version: "3"
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.13.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports: ["9200:9200"]
  kibana:
    image: docker.elastic.co/kibana/kibana:8.13.0
    ports: ["5601:5601"]
    depends_on: [elasticsearch]
```

```bash
docker compose up -d
```

Ship logs with **Filebeat** (lightweight log shipper) pointed at your
system's auth log:

```yaml
# filebeat.yml
filebeat.inputs:
  - type: log
    paths: ["/var/log/auth.log"]
output.elasticsearch:
  hosts: ["localhost:9200"]
```

## 3. Log sources that matter most for detection

| Source | Why it matters |
|---|---|
| **Authentication logs** (`/var/log/auth.log`, Windows Security log) | Every login attempt, success or failure |
| **Firewall/network logs** | Connections allowed/denied, unusual destinations |
| **DNS query logs** | Malware frequently uses DNS for command-and-control; unusual domains stand out |
| **Web server logs** | Requests, status codes, user agents — reveals scanning/exploitation attempts |
| **EDR/process logs** | What actually ran on an endpoint, parent/child process relationships |

## 4. Writing detection queries

In Kibana's Discover view (or raw Elasticsearch query DSL), a basic
failed-login-burst detection:

```json
GET /filebeat-*/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "message": "Failed password" } },
        { "range": { "@timestamp": { "gte": "now-5m" } } }
      ]
    }
  },
  "aggs": {
    "by_source_ip": {
      "terms": { "field": "source.ip", "size": 10 }
    }
  }
}
```

This finds every "Failed password" auth log line in the last 5 minutes,
grouped by source IP — a source IP with 50+ failures in 5 minutes is a
classic brute-force indicator, easy for a human to miss scrolling raw
logs, trivial for a SIEM to flag.

## 5. Alert design: reduce noise, catch signal

A SIEM that fires 500 alerts a day trains analysts to ignore all of them
(**alert fatigue**) — one of the most common real-world causes of missed
breaches. Good alert design:

```
Bad alert:  "Any failed login" -- fires constantly, mostly typos
Better:     "5+ failed logins from one source IP within 5 minutes,
             followed by a success" -- specific to actual brute-force-
             then-compromise pattern, rare enough to investigate every time

Bad alert:  "Any outbound connection to a new IP"
Better:     "Outbound connection to an IP with no reverse DNS, on a
             non-standard port, from a host that normally only makes
             outbound HTTPS to known SaaS domains"
```

The design principle: alert on **specific, correlated patterns**, not
single raw events — this is exactly what a SIEM's correlation engine is
for, versus a plain log grep.

## 6. Log analysis with the command line (when there's no SIEM yet)

Before reaching for a SIEM, these commands answer most first questions:

```bash
# Top source IPs by failed SSH attempts
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head

# Successful logins after a run of failures from the same IP -- possible
# successful brute force
grep -E "Failed password|Accepted password" /var/log/auth.log

# Web server: top requested paths that returned 404 -- reconnaissance/scanning
awk '{print $7, $9}' access.log | grep " 404" | sort | uniq -c | sort -rn | head
```

## 7. Retention and log integrity

- Retain logs long enough to investigate a breach discovered late — 90
  days minimum is common, longer for regulated environments (PCI-DSS
  requires at least a year, with 3 months immediately available).
- Ship logs to write-once or access-controlled storage so an attacker with
  host access can't retroactively edit what was already shipped.
- Time-synchronize every log source (NTP) — correlating events across
  systems is meaningless if their clocks disagree.

## Key terms

| Term | Meaning |
|---|---|
| **SIEM** | Security Information and Event Management — centralizes and correlates logs for detection |
| **Correlation** | Combining multiple individually-benign events into a meaningful alert |
| **Alert fatigue** | Analysts ignoring alerts because too many are low-value/false positives |
| **Log shipper** | Agent (Filebeat, Fluentd) that forwards logs from a host to a central store |
| **Brute force** | Repeated login attempts trying many passwords/usernames |
| **Retention period** | How long logs are kept before deletion/archival |

## Exercise

1. Stand up the ELK stack via the docker-compose file above.
2. Configure Filebeat to ship your own machine's auth log (or SSH log
   from a lab VM) into Elasticsearch.
3. In Kibana, build a Discover search that isolates failed-login events.
4. Write the aggregation query in section 4 and run it against your own
   shipped data (generate test failures with a few deliberate wrong-
   password SSH attempts against your lab VM).
5. Design one alert rule (in words, following section 5's format) for a
   pattern you think is high-signal, and explain why it wouldn't cause
   alert fatigue.

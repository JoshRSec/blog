---
title: "Building a Runtime Container Security Pipeline with Falco, Loki, and Traefik on TrueNAS"
date: 2026-02-07
summary: "How I deployed Falco, Sidekick, Loki, and Traefik on TrueNAS to create a secure, real-time container monitoring and alerting pipeline as part of my defense-in-depth strategy."
description: "A walkthrough of building a runtime container security stack using Falco, Sidekick, Loki, and Traefik on TrueNAS, including JSON log structuring, secure external access, and integration with Grafana."
slug: "truenas-falco-loki-container-security"
draft: false
aliases:
  - "/truenas-falco-loki-container-security"
categories:
  - "Homelab"
  - "Security"
tags:
  - "Falco"
  - "Loki"
  - "Traefik"
  - "TrueNAS"
  - "Docker"
  - "Runtime Security"
  - "Defense in Depth"
---

# A Guide to Strengthening Runtime Security with Falco, Sidekick, Loki, and Traefik

Security has always been a priority in my home lab, and over time I’ve built a fairly robust **defense‑in‑depth** strategy. At the perimeter, I run an IPS that inspects and filters traffic. Internally, I segment my network into VLANs and apply inspection rules to each one - including the VLAN where my Docker host lives. But even with those layers in place, I knew I was still missing visibility into one critical area:

**What’s happening inside my containers at runtime.**

My TrueNAS server runs sixteen Docker containers (not counting the monitoring stack itself), and while everything has been stable, I had no real insight into suspicious behavior inside those workloads. If a container suddenly spawned a shell, modified a sensitive file, or executed a package manager, I wouldn’t know until it was too late.

So I set out to build a proper runtime monitoring and alerting pipeline - something lightweight, open‑source, and fully under my control.

This write‑up walks through what I built, why I built it, and how you can replicate it.

## Why Runtime Security Matters in a Defense‑in‑Depth Strategy

Defense‑in‑depth is all about layering controls so that if one fails, another catches the issue. My perimeter IPS protects inbound and outbound traffic. VLAN segmentation limits lateral movement. But neither of those layers can see what’s happening *inside* a container.

That’s where runtime detection comes in.

Containers are isolated, but they’re not invincible. A misconfiguration, vulnerable dependency, or compromised process can lead to:

- Unexpected shells  
- Unauthorized file writes  
- Privilege escalation attempts  
- Suspicious network activity  
- Package manager execution  
- Tampering with binaries  

These behaviors rarely show up in application logs. I needed something that could observe system calls directly.

## Step 1 - Deploying Falco on TrueNAS

Falco is perfect for this job. It hooks into the kernel (via eBPF in my case) and inspects system calls in real time. When something suspicious happens, Falco raises an alert with rich metadata.

To run Falco on TrueNAS, I deployed it via Docker Compose with:

- `privileged: true`  
- `pid: host`  
- Access to `/proc`, `/etc`, `/sys`, and the Docker socket  

I also enabled the BPF probe, which is the recommended approach for modern kernels.

## Step 2 - Forwarding Alerts with Falco Sidekick

Falco emits events, but it doesn’t store them. That’s where **Falco Sidekick** comes in. Sidekick receives Falco alerts and forwards them to external systems - in my case, **Loki**.

I spent time tuning Sidekick’s output so Loki would receive clean, structured JSON:

- `LOKI_FORMAT=json`  
- `LOKI_TEXTFORMAT=all`  
- `LOKI_EXTRALABELS=container.name,proc.name,container.image.repository`  

These extra labels give me powerful filtering in Grafana without causing high cardinality issues.

Sidekick sends logs internally to:

```
http://loki:3100/loki/api/v1/push
```

No authentication needed - this traffic never leaves the Docker network.

### Why JSON Output Was Essential

One of the most important improvements I made during this project was switching Falco Sidekick from its default **text output** to **structured JSON**.

#### Text Output: Human‑Readable, Machine‑Unfriendly  
Falco’s default text output is designed for humans, not log aggregation systems. It produces long, descriptive strings like:

```
Warning Package management process launched in container | user=root process=apk ...
```

This looks fine in a terminal, but it’s extremely difficult for Loki to parse. Every field is embedded inside a single text blob, which means:

- Loki can’t reliably extract fields  
- Grafana can’t map values into columns  
- Queries become messy and error‑prone  
- Dashboards can’t group or filter by metadata  
- Alerts become harder to build  

#### JSON Output: Structured, Queryable, and Machine‑Friendly  
Switching to JSON changed everything. JSON gives each piece of data its own key/value pair:

```json
{
  "rule": "Package management process launched in container",
  "priority": "Error",
  "output_fields": {
    "proc.name": "apk",
    "container.name": "nzbget",
    "user.name": "root"
  }
}
```

This structure is perfect for Loki and Grafana because:

- Each field becomes queryable  
- Dashboards can group by rule, container, process, priority, etc.  
- LogQL can parse the event with a simple `| json`  
- Labels can be cleanly extracted without regex hacks  
- The data becomes consistent and predictable  

### Why Single‑Line JSON Matters  
Loki stores logs line‑by‑line. If JSON spans multiple lines, Loki treats each line as a separate log entry - which breaks parsing entirely.

That’s why I enabled:

```
LOKI_TEXTFORMAT=all
```

This forces Sidekick to flatten the entire JSON object into a single line, ensuring Loki receives complete, valid JSON every time.

#### In Summary  
Switching to JSON wasn’t just a formatting change - it was a foundational improvement that unlocked the full potential of Loki and Grafana. It turned Falco alerts from unstructured text into rich, queryable security events, making the entire monitoring pipeline far more useful and far easier to work with.

## Step 3 - Centralizing Logs with Loki

Loki is a log aggregation system designed to be efficient and scalable. It stores logs in a compressed format and indexes only labels, which makes it perfect for security telemetry.

I mounted persistent storage on TrueNAS so logs survive container restarts and configured Loki using a local config file.

Once Sidekick started pushing events, Loki immediately began storing structured Falco alerts.

## Step 4 - Securing Loki with Traefik

My Grafana instance runs on a different server, so Loki needed to be reachable externally. But Loki has **no built‑in authentication**, and exposing it directly would be a security risk.

To keep things secure, I placed Loki behind **Traefik** and added:

- **TLS via Let’s Encrypt**  
- **Basic Auth middleware**  

This ensures:

- Only authenticated users can query Loki  
- Only Sidekick can push logs  
- All external access is encrypted  
- Loki’s write API is never exposed publicly  

Here’s the exact Traefik label block I used:

```
traefik.enable=true
traefik.docker.network=traefik_infra_security
traefik.http.routers.loki.rule=Host(`loki.<internal dns>`)
traefik.http.routers.loki.entrypoints=https
traefik.http.routers.loki.tls.certresolver=letsencrypt
traefik.http.services.loki.loadbalancer.server.port=3100
traefik.http.middlewares.loki-auth.basicauth.users=${GRAFANA_USER}
traefik.http.routers.loki.middlewares=loki-auth
```

This gives me a secure, password‑protected Loki endpoint that Grafana can access remotely.

## Step 5 - Visualizing Alerts in Grafana

With Loki secured, I configured Grafana on my other server to use:

- URL: `https://loki.<internal dns>`  
- Basic Auth credentials  
- Loki as a datasource  

Grafana now pulls Falco events securely over HTTPS.

From there, I can now build dashboards showing:

- Alerts by rule  
- Alerts by container  
- Alerts by severity  
- MITRE ATT&CK tags  
- Process‑level activity  

This gives me real‑time visibility into suspicious behavior across all containers.

## Step 6 - Testing the Pipeline

To validate everything, I triggered safe Falco alerts such as:

- Running `apk update` inside a container  
- Spawning a shell inside a container  
- Touching a file in `/etc`  

Falco detected the behavior instantly, Sidekick forwarded it, Loki stored it, and Grafana displayed it.

Seeing the full pipeline working end‑to‑end was incredibly satisfying.

## The Final Compose File

```
networks:
  traefik_infra_security:
    external: true
services:
  falco:
    image: falcosecurity/falco:latest
    container_name: falco
    privileged: true
    pid: host
    restart: unless-stopped
    environment:
      - FALCO_BPF_PROBE=enabled
    depends_on:
      - sidekick
    volumes:
      - /var/run/docker.sock:/host/var/run/docker.sock:ro
      - /proc:/host/proc:ro
      - /etc:/host/etc:ro
      - /sys/kernel/tracing:/sys/kernel/tracing:ro
      - /sys/kernel/debug:/sys/kernel/debug:ro
      - ./falco:/etc/falco
    networks:
      - traefik_infra_security
  sidekick:
    image: falcosecurity/falcosidekick:latest
    container_name: sidekick
    environment:
      - LOKI_ENABLED=true
      - LOKI_HOSTPORT=http://loki:3100
      - LOKI_ENDPOINT=/loki/api/v1/push
      - LOKI_MINIMUMPRIORITY=debug
      - LOKI_FORMAT=json
      - LOKI_TEXTFORMAT=all
      - LOKI_EXTRALABELS=container.name,proc.name,container.image.repository
    depends_on:
      - loki
    restart: unless-stopped
    networks:
      - traefik_infra_security
  loki:
    image: grafana/loki:3.6.5
    container_name: loki
    command: -config.file=/etc/loki/local-config.yaml
    restart: unless-stopped
    environment:
      - PUID=10001
      - PGID=10001
    volumes:
      - ./loki-data:/loki
      - ./loki-config:/etc/loki
    networks:
      - traefik_infra_security
    labels:
      - traefik.enable=true
      - traefik.docker.network=traefik_infra_security
      - traefik.http.routers.loki.rule=Host(`loki.<internal dns>`)
      - traefik.http.routers.loki.entrypoints=https
      - traefik.http.routers.loki.tls.certresolver=letsencrypt
      - traefik.http.services.loki.loadbalancer.server.port=3100
      - traefik.http.middlewares.loki-auth.basicauth.users=${GRAFANA_USER}
      - traefik.http.routers.loki.middlewares=loki-auth
```

## What This Adds to My Defense‑in‑Depth Strategy

With this pipeline in place, I now have:

- **Perimeter protection** via IPS  
- **Network segmentation** via VLANs  
- **Runtime behavioral detection** via Falco  
- **Centralized log storage** via Loki  
- **Secure remote access** via Traefik  
- **Dashboards and analysis** via Grafana  

Each layer reinforces the others. If something slips past the perimeter or moves laterally inside the network, Falco will catch it at runtime. If Falco fires an alert, Loki stores it, and Grafana visualizes it.

This is exactly what defense‑in‑depth is supposed to look like.

## Final Thoughts

This project transformed my TrueNAS Docker environment from a black box into a monitored, observable, and secure platform. Falco gives me behavioral detection, Sidekick handles forwarding, Loki stores everything efficiently, and Traefik ensures secure access.

If you’re running containers - especially more than a handful - this setup is absolutely worth the effort. It’s lightweight, open‑source, and incredibly powerful.

### Next Steps
Now that the core pipeline is in place, I’m letting it run for a few days to build a clean baseline of “normal” activity. Once that’s established, these are the next areas I’ll be focusing on:

**Rule Refinement**

- Tune Falco’s default rules to reduce noise
- Add custom rules tailored to my environment
- Identify high‑value detections vs. low‑value chatter
- Adjust Sidekick filtering so Loki only receives meaningful events

**Alerting**

- Refine severity tiers (info, warning, critical)
- Configure alert routing (email, Slack, webhook, etc.)

**Dashboard Creation**

- Create a unified Grafana dashboard for Falco → Loki → Traefik
- Add panels for container activity, syscall anomalies, and network events
- Build a panel showing recent Falco alerts in timeline form

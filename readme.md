# PJ Market Valuation & Salvage Guide

A centralized, cloud-hosted vehicle valuation and market alert system designed for live market pricing, salvage buy-in calculations, and automated background monitoring.

## System Architecture

This system uses a centralized "Cloud Brain" to eliminate split-brain data issues. Both the mobile and desktop clients act as remote controls that read from and write to a single source of truth hosted on Cloudflare.

```mermaid
graph TD
    subgraph "Client Interfaces"
        Phone["📱 Phone Web App<br/>(GitHub Pages / HTML+JS)"]
        Java["💻 Desktop App<br/>(IntelliJ Java .exe)"]
    end

    subgraph "The Cloud Engine (Cloudflare)"
        Worker{"⚙️ Cloudflare Worker<br/>(pj-val-proxy)"}
        KV[("🗄️ KV Database<br/>(CAR_ALERTS)")]
        Cron(("⏱️ Background Cron<br/>(Runs every 30 mins)"))
    end

    subgraph "External Targets"
        DoneDeal["🚗 DoneDeal API<br/>(Market Data)"]
        Resend["📧 Resend API<br/>(Email Dispatcher)"]
    end
    
    Inbox["📥 Gmail Inbox"]

    Phone <-->|HTTP GET/POST<br/>Saves Filters & Runs Scans| Worker
    Java <-->|HTTP GET/POST<br/>Saves Filters & Runs Scans| Worker

    Worker <-->|Reads/Writes active filters<br/>& Sent-Ad history| KV
    Cron -->|Wakes up Worker automatically| Worker

    Worker -->|Sends strict search parameters| DoneDeal
    DoneDeal -->|Returns JSON vehicle listings| Worker

    Worker -->|If brand new Ad ID found| Resend
    Resend -->|Delivers HTML Alert| Inbox

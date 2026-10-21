# Systems Architecture: Latency & CDN Resiliency Blueprint

## 1. System Topology Overview

```mermaid
graph TD
    User([Browser Client]) -->|Request &lt; 25ms| CDN[Global Edge CDN - jsDelivr / Cloudflare]
    CDN -->|Cache Hit| FastResp[Compressed Brotli Response]
    CDN -->|Cache Miss| OriginBackend[CDN Origin Git Puller]
    OriginBackend -->|Raw Git Trees| GitHub[GitHub Repository]
    User -->|Local Cache| SWR[SWR Browser Memory / Disk]
```

## 2. Component Specifications

- **Edge Tier:** jsDelivr multi-CDN mesh (300+ PoPs globally) providing Brotli compression and SSL termination.
- **Client Tier:** SWR client library handling client-side memoization and background revalidation.
- **Renderer:** Shiki TextMate grammar syntax highlighter running non-editable code rendering.

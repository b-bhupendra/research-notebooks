# Modern Systems Architecture: Latency & CDN Resiliency

> Deep-dive analysis into sub-25ms edge delivery, zero-rate-limit client architectures, and distributed caching topologies for technical documentation and notebook rendering.

---

## 1. The Origin Bottleneck Problem

Serving raw technical content and computational artifacts directly from origin repositories (such as `raw.githubusercontent.com`) introduces three fundamental bottlenecks:

1. **Geographic RTT & High TTFB:** Centralized US origins incur 180ms – 550ms Time To First Byte (TTFB) for global audiences.
2. **Rate-Limiting Throttles:** GitHub unauthenticated IP limits throttle requests at 60 req/hour, breaking high-traffic websites and client-side web apps.
3. **MIME-Type & CORS Restrictions:** Origin storage forces restrictive headers like `text/plain`, corrupting SVGs, WebAssembly binaries, and dynamic JSON schemas.

---

## 2. Multi-CDN Edge Mesh Architecture

```text
[ Browser Client ]
       │
       ▼ (Edge Request < 25ms TTFB)
[ Global CDN PoP (Cloudflare / Fastly / Akamai) ]
       │
   Cache Hit? ──► YES ──► Serve Brotli/Gzip Compressed Asset
       │
       NO (Origin Fetch)
       ▼
[ CDN Origin Cluster ] ──(Git Tree Ingestion)──► [ GitHub Repository ]
```

### Key Advantages of Edge Decoupling

- **Immutable Pinning:** Tag-pinned requests (`@v1.0.0` or `@commit_sha`) are cached permanently at edge nodes for up to 1 year.
- **Auto-Negotiated Compression:** Edge nodes convert raw assets into Brotli (`br`), saving 25-35% bandwidth over vanilla Gzip.
- **High-Concurrency Resilience:** Even sudden traffic spikes of tens of thousands of requests are absorbed entirely at the edge without a single hit to GitHub's API rate limits.

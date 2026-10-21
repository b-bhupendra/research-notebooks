# GitHub Directory Design & CDN Comparison Plan

> **Architectural blueprint designed for fast retrieval, zero-rate-limit resilience, and clean, non-editable Jupyter/IPython rendering.**

---

## 1. Optimal Public GitHub Directory Structure

```text
research-notebooks/
├── manifest.json                  # ⚡ SINGLE-REQUEST INDEX: Project bundles, slugs, tags, dates, CDN URLs
├── README.md                      # Architecture blueprint & CDN benchmarks
├── .github/workflows/
│   └── publish-notify.yml         # CDN purge & receiving-site webhook notification
│
├── <project-slug>/                # 📦 UNIFIED PROJECT BUNDLE (e.g. systems-architecture)
│   ├── article.md                 # Technical blog post with frontmatter
│   ├── architecture.md            # System design & component blueprints
│   ├── notebook.ipynb             # Interactive Jupyter Notebook (.ipynb)
│   └── assets/                    # Project-specific diagrams, SVGs & charts
└── ...
```

---

## 2. The Single-Request Manifest Pattern (Zero-Crawling)

**Never use the GitHub REST API to recursively crawl directory trees (e.g. `/contents/`).**

- Standard unauthenticated GitHub API limits are strictly capped at **60 requests/hour** per IP address.
- Instead, commit a generated `manifest.json` at the root of the repository.
- The client web application executes **exactly one HTTP GET request** (cached at the edge) to fetch all post metadata, slugs, categories, tags, and reading times instantly.

---

## 3. Direct GitHub Raw vs. CDN Edge (jsDelivr / Cloudflare)

### Live In-Browser Benchmark: How Much Time Does a CDN Save?

| Criteria | Direct GitHub (`raw.githubusercontent.com`) | CDN Edge (e.g., jsDelivr / Cloudflare) |
| :--- | :--- | :--- |
| **Edge Latency (TTFB)** | 180ms – 550ms (Variable, origin in US) | **< 25ms** (300+ Global Edge PoPs) |
| **Rate Limits** | Strict IP throttle; 60/hr API limit | **Virtually unlimited** high-traffic throughput |
| **CORS & MIME Types** | Forces `text/plain`; risky for SVGs/HTML | Full `Access-Control-Allow-Origin: *` + correct MIME |
| **Cache Invalidation** | 5–15 min delayed caching | Instant via Git Tag/Commit SHA: `@hash/file` |
| **Compression** | Standard Gzip | **Brotli (`br`) + Gzip** auto-negotiated |

---

## 4. Best Free CDNs for Public GitHub Technical Blogs

### 1. jsDelivr (Recommended · 100% Free)
*Zero-configuration GitHub mirror. Powered by Cloudflare, Fastly, and Akamai multi-CDN mesh.*
- No sign-up or API key needed
- Automatic Brotli & Gzip compression
- Built-in version pinning (`@v1.0.0` or `@commit_sha`)
- Zero rate-limiting on public repositories
- **Endpoint Pattern:** `https://cdn.jsdelivr.net/gh/:user/:repo@:tag/:file`

### 2. Cloudflare Free Tier (Enterprise-Grade Free)
*Industry gold-standard edge network. Unlimited bandwidth across 330+ worldwide cities.*
- Unlimited bandwidth with zero overage fees
- Custom domain support (e.g., `cdn.yourdomain.com`)
- Automatic HTTPS, HTTP/3, and DDoS protection
- Works seamlessly with Cloudflare Pages & Workers

### 3. Statically.io (Open Source CDN)
*Specialized multi-CDN built specifically for GitHub, GitLab, and open-source static assets.*
- On-the-fly WebP image transcoding
- Fast multi-CDN edge network
- Full CORS support out of the box
- **Endpoint Pattern:** `https://cdn.statically.io/gh/:user/:repo/:branch/:file`

### 4. GitHub Pages (Fastly Edge · Built into GitHub)
*Automated hosting for public repositories backed by Fastly edge caching.*
- Custom CNAME domain support
- Automated GitHub Actions deploy pipeline
- 100GB/month soft bandwidth allowance

---

## 5. How Public CDNs Natively Ingest & Cache from GitHub

1. **Edge Cache Lookup:** When a user requests `https://cdn.jsdelivr.net/gh/user/repo@v1.2.0/file.py`, the nearest Edge Point of Presence (Cloudflare/Fastly/Akamai) checks its local edge cache. On a hit, it serves with `< 20ms` TTFB.
2. **Origin Git Pull on Cache Miss:** If the file is not yet cached, the CDN edge forwards the request to the CDN origin backend. The backend clones/pulls the raw Git tree directly from GitHub using internal high-speed pipes.
3. **Brotli / Gzip Optimization:** The CDN automatically compresses the raw file with modern Brotli (`br`), minifies it where applicable, and attaches proper headers (`Content-Type`, `Access-Control-Allow-Origin: *`).
4. **Global Edge Propagation:** The optimized asset is replicated across all 300+ global edge locations.
5. **Cache Invalidation & Immutability:** Pinning by tag (`@v1.0.0`) or commit hash (`@commit_sha`) makes the URL immutable (cached for up to 1 year). If using a branch (`@main`), it caches for 12 hours and can be purged instantly via the free purge API (`https://purge.jsdelivr.net`).

---

## 6. Research: Best Libraries for Markdown & Sleek Code Highlighting

| Library / Engine | Highlighting Mechanism | Visual Fidelity | Bundle Weight | Best Fit & Key Pros |
| :--- | :--- | :--- | :--- | :--- |
| **Shiki / Rehype-Pretty-Code** | TextMate Grammars (VS Code Engine) | **Highest (10/10)** - True VS Code themes | ~100KB – 250KB (WASM grammar) | Dual dark/light themes without flash, line highlighting (`{3-5}`), word highlights, git diff lines. The gold standard for modern tech blogs. |
| **PrismJS** | Regex-based tokenization | Medium (7.5/10) - Good classic look | Ultra-light (~15KB) | Fast, zero WASM dependencies, vast plugin ecosystem (line numbers, copy button). Can occasionally mis-parse complex JSX/TypeScript generics. |
| **Highlight.js** | Heuristic regex with auto-detection | Medium (7/10) - Standard IDE style | ~35KB core + languages | Supports 190+ languages out of the box with automatic language detection. Lacks built-in line highlighting. |
| **React-Markdown + Remark-GFM** | Unified AST Pipeline | High (9/10) - Custom component mapping | Lightweight (~25KB) | Replaces standard markdown tags with custom sleek React components (MUI tables, copy buttons, custom code cards, LaTeX KaTeX). |
| **Prettier Standalone** | AST Auto-Formatting / Prettifying | Flawless indentation & wrap | Medium (~120KB for babel/python) | Formats and normalizes messy un-indented code blocks directly in browser memory before highlighting. |

---

## 7. Sleek Non-Editable Presentation Standard

- **Tokenized Multi-Language Highlighting:** Custom lexical tokenization for Python, SQL, TypeScript, Bash, and JSON with VS Code dark palette.
- **Gutter Discipline:** Subtle line numbers gutter with clean border separation and one-click copy button.
- **Interactive DataFrame Grids:** Renders Pandas HTML tables as sleek, horizontally scrollable data grids with zebra rows.
- **Vector & Lightbox Visualizations:** Renders Matplotlib/Seaborn SVG & PNG outputs with dark backdrops, responsive scaling, and full-screen lightbox viewing.
- **Structured Blockquotes & Headers:** Clean typographical kicker lines, quotes with primary color accents, and zero candy badges.

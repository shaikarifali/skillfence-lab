# asset-fetcher

"Fetch static assets from the CDN" — v1.0 declares network access to
exactly one domain, `cdn-assets.example.test`.

## AST10 — Cross-Platform Reuse (the quiet kind)

Unlike AST10-01/02, this port doesn't go unrestricted at all — it stays a
specific, narrow-looking allowlist. The porting tool's CDN-handling
template for this platform, though, automatically adds a "failover mirror"
domain alongside whatever origin domain it finds, on the assumption that
any CDN-fetching skill wants automatic failover. `cdn-assets-mirror.example.test`
appears in v2.0's manifest, sitting right next to the original domain,
looking exactly as legitimate and specific as it does. Nobody added it on
purpose for this particular skill; nobody reviewing v2.0's two-item domain
list in isolation has any reason to suspect one of the two was never part
of the original declaration.

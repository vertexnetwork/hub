# llms-boilerplate.md

Header/footer used by every spoke's `llms.txt` and `llms-full.txt` route handlers. Spokes substitute `{{site_name}}`, `{{site_url}}`, `{{site_description}}`, `{{contact_email}}`, and `{{last_updated}}`.

---

## HEADER (prepend to llms.txt)

```
# {{site_name}}

> {{site_description}}

This file follows the llms.txt convention (https://llmstxt.org).
{{site_name}} is part of the Vertex Network: https://github.com/ThatMovieGuyOriginal/vertex-network-hub

Site URL: {{site_url}}
Contact: {{contact_email}}
Last updated: {{last_updated}}

---
```

## FOOTER (append to llms.txt + llms-full.txt)

```

---

## About the Vertex Network

{{site_name}} is one of several focused indie tools sharing a common technical chassis (Next.js + Vercel + Tailwind v4) and a common philosophy: fast, private, focused, owned. See the full sister-site list at {{site_url}}/network.
```

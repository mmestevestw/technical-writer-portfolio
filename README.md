# Technical Writing Portfolio — Matheus Esteves

Senior Technical Writer with experience documenting enterprise software: REST APIs, microservice architectures, and installation/configuration guides for complex, multi-component systems.

This repo contains writing samples that demonstrate my documentation approach across a few common technical-writing deliverables. To keep this portfolio public and free of any employer-confidential material, the samples below are built around **Snipe-IT**, a real, open-source IT asset management platform — but they're written entirely by me, from scratch, based on my own research into its public API and installation process. They reflect the same structure, depth, and style I use for proprietary API and platform documentation in my day job.

## What's here

| Sample | What it shows |
|---|---|
| [`api-reference.md`](api-reference.md) | A REST API reference doc (resource model, operations, request/response examples, error handling) — the kind of deliverable I produce for engineering-facing API documentation. |
| [`installation-configuration-guide.md`](installation-configuration-guide.md) | A Docker-based install/configuration guide, including prerequisites, step-by-step setup, and common troubleshooting — modeled on the install guides I write for platform teams. |
| [`product-overview.md`](product-overview.md) | A product overview doc aimed at a mixed technical/business audience — architecture summary, key features, and module breakdown. |
| [`docs-as-code-workflow.md`](docs-as-code-workflow.md) | A case study template describing a docs-as-code migration (Git-based workflow, automated style linting with Vale, CI-driven publishing). Fill in with your own project specifics — written generically on purpose so it can describe process and impact without disclosing any employer's internal systems. |

## About my approach

- **Docs-as-code**: I write and maintain documentation in Markdown/AsciiDoc, version-controlled alongside the product, with automated linting (Vale) and CI-driven builds and publishing.
- **API documentation**: I work from OpenAPI/Swagger specs where available, scripting reference-doc generation in Python to keep docs synced with the spec, and hand-write the surrounding conceptual and example content.
- **Structured authoring**: For larger doc sets, I favor a modular structure (overview → concepts → tasks → reference) so content can be reused and updated independently.
- **Collaboration**: I work directly with engineers and product owners to verify technical accuracy, and mentor junior writers on style consistency and doc architecture.

## Contact

mmesteves13@gmail.com · https://www.linkedin.com/in/matheus-marques-esteves-b4207234/ · Petrópolis, Rio de Janeiro, Brazil

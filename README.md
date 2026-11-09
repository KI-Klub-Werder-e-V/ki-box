# KI-Box: a private German-language AI workstation for a 16 GB PC without a GPU

English · [Deutsch](README.de.md)

**KI-Box is a planned one-command installer that turns a small Linux PC with 16 GB RAM, no GPU and a roughly six-year-old CPU into a private AI workstation with chat, German document search with sources, user accounts and encrypted backups.** It packages existing open components (Ollama, Open WebUI, Caddy, restic) in one tested low-memory profile. It is a project of [KI Klub Werder e.V.](https://ki-klub-werder.de), a non-profit AI club in Werder (Havel), Germany.

> **Status (November 2026): first build available.** [**KI-Box Harbor**](https://github.com/KI-Klub-Werder-e-V/ki-box-harbor) is a stripped-down fork of Harbor with only Ollama and Open WebUI and the defaults below. It has not yet been measured on the target hardware. This repository holds the full design in [`docs/spec.de.md`](docs/spec.de.md) (German); backup, restore and HTTPS are still to come.

## What KI-Box is

| | |
|---|---|
| **Target hardware** | Linux PC, 16 GB RAM, no GPU, CPU about 6 years old with AVX2, SSD |
| **Operating system** | Ubuntu Server 24.04 LTS |
| **Target users** | AI enthusiasts running a shared box at home, in a club or a small business |
| **Network** | Local network only, no internet exposure in v1 |
| **Language focus** | German chat, German document retrieval |
| **Licence** | Apache-2.0 for KI-Box's own code; bundled components keep their own licences |
| **Cloud dependency** | None at runtime; the internet is needed for installing and updating |

## Planned features

- **One-command install** that can be re-run safely after an interruption
- **Local chat** with a small open-weight model chosen by a German test set
- **German document search** with hybrid retrieval and cited sources
- **OCR on demand** for scanned PDFs, run once instead of as a permanent service
- **User accounts and groups**, with document access per group
- **Encrypted nightly backups** with a tested restore path
- **Safe updates**: pinned versions, a backup before every update, a smoke test after it
- **OpenAI-compatible API** on the local network for your own experiments

## Components

| Job | Component | Licence |
|---|---|---|
| Model runtime | [Ollama](https://github.com/ollama/ollama) | MIT |
| Chat, accounts, retrieval | [Open WebUI](https://github.com/open-webui/open-webui) | Open WebUI License (not OSI-approved) |
| OCR for scans | [OCRmyPDF](https://github.com/ocrmypdf/OCRmyPDF) | MPL-2.0 |
| HTTPS | [Caddy](https://github.com/caddyserver/caddy) | Apache-2.0 |
| Encrypted backup | [restic](https://github.com/restic/restic) | BSD-2-Clause |
| Embeddings | [BGE-M3](https://huggingface.co/BAAI/bge-m3) | MIT |
| Chat model candidates | [Gemma 4](https://huggingface.co/google/gemma-4-E4B-it), Qwen 3.5 | Apache-2.0 |

Open WebUI's licence restricts removing its branding above 50 users. KI-Box uses the official image unchanged and keeps the branding. KI-Box as a whole is therefore not "100% open source" in the OSI sense; only its own code is.

## Memory plan for 16 GB

| Item | Memory | Basis |
|---|---|---|
| Linux and Docker | about 1.5 GB | estimate |
| Open WebUI | about 1.5 GB | estimate |
| BGE-M3 embeddings | 1.2 GB | Ollama model size |
| Chat model `gemma4:e4b-it-qat` | 6.1 GB | Ollama model size |
| Context, cache, reserve | about 5.7 GB | remainder |

Ollama sizes are download sizes (November 2026); actual RAM use is higher and depends on context length. The profile therefore caps context at 8,192 tokens and handles one request at a time. Smaller fallbacks are `gemma4:e2b-it-qat` (4.3 GB) and `qwen3.5:4b` (3.3 GB).

## FAQ

### Can you run a useful local LLM on 16 GB RAM without a GPU?

Yes, with limits. A small quantised model fits in 16 GB, but CPU-only output is slow: third-party reports put an Intel N100 at about 6–9 tokens per second on an 8B model. Expect one active request at a time. That suits enthusiasts and asynchronous questions, not a team that needs instant answers. KI-Box will publish its own measurements before release.

### Which model is best for German on a small PC?

Undecided. KI-Box will choose between Gemma 4 E4B, Gemma 4 E2B and Qwen 3.5 4B with a German test set built on real municipal bylaws, and publish the results.

### Why does German document search need special setup?

Two Open WebUI defaults do not suit German. The default embedding model is `all-MiniLM-L6-v2`, which is English-oriented. Its keyword search splits text only on whitespace, so "Hundesteuer," and "Hundesteuer" are different terms and compounds such as "Hundesteuersatzung" are not decomposed (checked in v0.13.3). KI-Box switches embeddings to BGE-M3 and will measure the keyword issue.

### How is KI-Box different from Harbor, Open WebUI or AnythingLLM?

| | KI-Box (planned) | [Harbor](https://github.com/av/harbor) | [Open WebUI](https://github.com/open-webui/open-webui) alone | [AnythingLLM](https://github.com/Mintplex-Labs/anything-llm) |
|---|---|---|---|---|
| **Purpose** | One fixed profile for 16 GB, CPU only | Toolkit to launch many local LLM services | Chat and retrieval application | Document-focused AI application |
| **German retrieval preset** | Yes | No | Manual | Manual |
| **Backup and restore included** | Yes | Not a headline feature | Manual | Manual |
| **Status** | Specification | Released | Released | Released |

Want to try many backends and services? Use Harbor today. KI-Box is for one tested setup that stays maintained. KI-Box now builds on Harbor as the stripped-down fork [KI-Box Harbor](https://github.com/KI-Klub-Werder-e-V/ki-box-harbor).

### Does any data leave the machine?

Not by design. Open WebUI's image disables telemetry, and KI-Box will set offline mode and turn off web search. Release acceptance includes capturing outbound traffic to prove it. Local operation does not by itself make you GDPR compliant; the operator remains the data controller.

### Does KI-Box work with Model Context Protocol (MCP)?

Open WebUI can connect to MCP servers over Streamable HTTP. The planned link to [MarieKondoForAgents](https://github.com/KI-Klub-Werder-e-V/MarieKondoForAgents) is deferred because its HTTP mode requires an OAuth 2.1 authorization server. In v1, cleaned documents are uploaded by hand.

## Install

The first build lives in [KI-Box Harbor](https://github.com/KI-Klub-Werder-e-V/ki-box-harbor):

```bash
curl -fsSL https://raw.githubusercontent.com/KI-Klub-Werder-e-V/ki-box-harbor/main/install.sh | bash
harbor up
```

## Roadmap

1. ~~Decide whether to build on Harbor or stand alone~~ Built on Harbor: [KI-Box Harbor](https://github.com/KI-Klub-Werder-e-V/ki-box-harbor)
2. Measure speed, memory and German answer quality on a 16 GB CPU-only PC
3. Build the installer, backup, restore and update commands
4. Pilot for four weeks, then release v1.0.0

## Contributing

The most useful contribution right now is a benchmark: your CPU model, RAM, model tag and tokens per second. Please [open an issue](https://github.com/KI-Klub-Werder-e-V/ki-box/issues).

## Licence

[Apache-2.0](LICENSE). A project of [KI Klub Werder e.V.](https://ki-klub-werder.de). Related projects: [OpenVerein](https://github.com/KI-Klub-Werder-e-V/openverein), [MarieKondoForAgents](https://github.com/KI-Klub-Werder-e-V/MarieKondoForAgents), [Werdi](https://github.com/KI-Klub-Werder-e-V/werdi), [Kommentarwache](https://github.com/KI-Klub-Werder-e-V/kommentarwache).

*Facts last checked: 9 November 2026.*

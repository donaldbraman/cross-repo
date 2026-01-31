# Sibling Repository Map

**Applies to:** All repositories
**Parent directory:** `~/Documents/GitHub/`

This guide helps Claude Code agents understand the local repository ecosystem so they can find shared utilities, related projects, and cross-repo dependencies.

## Repositories

| Repository | Path | Description |
|------------|------|-------------|
| [auth-utils](https://github.com/donaldbraman/auth-utils) | `~/Documents/GitHub/auth-utils` | Shared authentication infrastructure for LLM providers and Google/Zotero APIs |
| [cross-repo](https://github.com/donaldbraman/cross-repo) | `~/Documents/GitHub/cross-repo` | Central shared Claude Code instructions, agents, skills, commands, and guides |
| [admin-assistant](https://github.com/donaldbraman/admin-assistant) | `~/Documents/GitHub/admin-assistant` | Google Workspace plugins and project tracking for administrative task automation |
| [cite-assist](https://github.com/donaldbraman/cite-assist) | `~/Documents/GitHub/cite-assist` | Citation management integrating Zotero with semantic search via Qdrant and AI models |
| [write-assist](https://github.com/donaldbraman/write-assist) | `~/Documents/GitHub/write-assist` | Multi-LLM legal academic writing assistance using Claude, Gemini, and ChatGPT |
| [jil-seminar](https://github.com/donaldbraman/jil-seminar) | `~/Documents/GitHub/jil-seminar` | Course materials and data collection tools for the Justice Innovation Lab seminar |
| [pin-citer](https://github.com/donaldbraman/pin-citer) | `~/Documents/GitHub/pin-citer` | Automated pin citation generator for law review articles using cite-assist |
| [prosecutors-paradox](https://github.com/donaldbraman/prosecutors-paradox) | `~/Documents/GitHub/prosecutors-paradox` | Interactive simulation of differential arrest rates and sentencing policies |
| [criminal-law](https://github.com/donaldbraman/criminal-law) | `~/Documents/GitHub/criminal-law` | Modern open-access criminal law casebook built with Quarto |
| [zothein](https://github.com/donaldbraman/zothein) | `~/Documents/GitHub/zothein` | Agentic research assistant integrating HeinOnline research with Zotero |

## Key Dependencies

- **auth-utils** is a shared dependency used by cite-assist, write-assist, admin-assistant, and zothein
- **cite-assist** provides citation services used by pin-citer and write-assist
- **cross-repo** provides shared Claude Code configuration used by all repos

---

*Last updated: 2026-01-31*

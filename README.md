# border.bot agent skills

Agent skills for [border.bot](https://border.bot), which classifies products to HS/HTS codes and calculates the duties, taxes, fees and landed cost of cross-border parcels, over its [MCP server](https://border.bot/mcp) or [REST API](https://border.bot/docs).

## Install

```bash
npx skills add borderbot-dev/skills
```

## Skills

| Skill | What it does |
| --- | --- |
| [landed-cost](skills/landed-cost/SKILL.md) | Get HS/HTS codes, import duties, taxes, fees and the landed cost of a cross-border parcel or cart from border.bot, over its MCP server or REST API, with the legal basis and source of every line. Use when a shopper, merchant or developer asks what an import will cost, which tariff code applies, how US Section 232 or 301 duties stack, where a product was made, whether goods are restricted, what an agency such as CPSC needs filed, or whether a buyer is on a denied-party list. |

border.bot also serves these files at https://border.bot/.well-known/agent-skills/index.json. This repository is generated from border.bot's source, so a change made here is overwritten: send fixes and questions to info@border.bot.

# Digital Garden by Alicia

A free interactive 3D flower garden in your browser. Explore four seasons, change the light and colours, listen to nature sounds, and make a flower photograph or a short clip.

**[Visit the garden](https://digitalgarden.si/)** · [中文介绍](README.zh-CN.md) · [What you can do](https://digitalgarden.si/virtual-garden/)

![Spring lily-of-the-valley flowers in Digital Garden by Alicia](https://digitalgarden.si/marketing/spring.jpg)

## Explore and photograph

- Choose spring, summer, autumn or winter, by day or night.
- Move closer to a flower, adjust the wind and recolour plants and light.
- Choose a frame and focus for a still photo, or record a clip up to 10 seconds.
- Save your capture or use native file sharing on a supported browser.
- No account or installation is required. Preferences and captures stay on your device until you choose to share them.

This is an explorable botanical artwork. It is distinct from the personal knowledge-base meaning of “digital garden.”

## Pick a season

| Scene | Plants to look for | Open |
| --- | --- | --- |
| Spring meadow | Lily of the valley, grape hyacinth and fern | [Day](https://digitalgarden.si/play/?season=spring&night=0&lang=en) · [Night](https://digitalgarden.si/play/?season=spring&night=1&lang=en) |
| Summer garden | Lavender, iris and dandelion | [Day](https://digitalgarden.si/play/?season=summer&night=0&lang=en) · [Night](https://digitalgarden.si/play/?season=summer&night=1&lang=en) |
| Autumn field | Wheat, sunflower and cosmos | [Day](https://digitalgarden.si/play/?season=autumn&night=0&lang=en) · [Night](https://digitalgarden.si/play/?season=autumn&night=1&lang=en) |
| Winter snowfield | Snowdrop, hellebore and camellia | [Day](https://digitalgarden.si/play/?season=winter&night=0&lang=en) · [Night](https://digitalgarden.si/play/?season=winter&night=1&lang=en) |

## Made with Houdini and Three.js

Alicia authored the plants in Houdini. GLB assets become an instanced flower scene in Three.js, with live wind, lighting and camera controls. WebGPU rendering has a WebGL2 compatibility mode.

On a phone, a smaller circular patch keeps nearby flowers available for a full horizontal orbit. Pinch to zoom from 1× to 5×; movement across the ground is limited. Device and browser performance varies. Sound starts after a user gesture.

[Read the making-of notes](https://digitalgarden.si/about/#making-of) · [Controls, audio and capture guide](https://digitalgarden.si/guide/)

## MCP notes for AI assistants

Digital Garden also provides a public, read-only MCP endpoint (Streamable HTTP, no authentication):

`https://mcp.digitalgarden.si/mcp`

Tools: `search_articles`, `get_article`, `list_topics`, `list_recent_articles`. Search or read Alicia’s English and Chinese notes on Houdini, three.js/WebGPU, glTF, OpenUSD and AI workflows, with canonical source URLs for citation.

[Connection guide and examples](https://digitalgarden.si/agents/) · [Technical blog](https://digitalgarden.si/blog/)

This repository contains showcase documentation. The hosted MCP service is remote-only; this repository does not contain an installable server package.

## Sharing and commercial use

Personal exploration is free. For sharing your captures, credit **Digital Garden by Alicia** and link to **https://digitalgarden.si/**. Winter clips containing the snow recording also need its third-party attribution; see the [usage guide](https://digitalgarden.si/terms/) and [credits](https://digitalgarden.si/credits/).

Commercial website licensing and custom garden commissions are available by inquiry. Scope and third-party rights are described on the [licensing](https://digitalgarden.si/licensing/) and [custom garden](https://digitalgarden.si/custom-gardens/) pages.

This repository presents the artwork and links to the live experience. It does not distribute the garden runtime, plant source files or an open-source license for those materials. See [rights and sources](RIGHTS.md).

**Official website:** [digitalgarden.si](https://digitalgarden.si/) · **Artist:** [Alicia](https://digitalgarden.si/about/)

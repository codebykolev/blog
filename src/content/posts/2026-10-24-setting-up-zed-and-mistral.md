---
title: Setting up Zed and Mistral
author: Code by Kolev
pubDatetime: 2026-10-26T16:30:32.798Z
slug: zed-mistral-setup
featured: false
draft: false
tags:
  - Zed
  - AI
  - Mistral
description: Documenting my journey setting up Zed and Mistral Large 4 (EU)
keywords:
  - zed
  - mistral
---

I'd been putting off setting up a better AI dev environment for a while, but with the release of Mistral Large 4 (ML4) I figured it was high time.

## Picking my stack

Starting with a minimal stack, we only need a couple of pieces: an editor and an AI provider.

### Which editor to choose?

There were really a couple of options here:

- [VSCode](https://code.visualstudio.com) — The Standard
- [Cursor](https://cursor.com)
- [WebStorm](https://www.jetbrains.com/webstorm)
- [Vim](https://www.vim.org)
- [Zed](https://zed.dev)
- [Helix](https://helix-editor.com)
- [Cline](https://cline.bot/ide)
- [Continue](https://continue.dev)

Wanting an open source option disqualified Cursor and WebStorm. Wanting to learn something new disqualified VSCode. Continue was **dis**continued (haha, pun intended). Cline is an extension. So we were left with:

- Vim
- Zed
- Helix

Never got too used to Vim, and Helix is super interesting, but would require a bit of time to learn. That left only Zed, which felt like a good choice, given it is now the fastest-growing standalone option (Brave AI said so).

### Which AI hosting provider to choose

Many options to choose from here:

- [RunPod](https://runpod.io)
- [DigitalOcean](https://digitalocean.com)
- [Replicate](https://replicate.com)
- [Modal](https://modal.com)
- [Together AI](https://together.ai)
- ...

But I wanted something with EU residency, and maybe even EU-based, so I looked at EU options:

- [Nebius](https://nebius.com)
- [OVHcloud](https://ovhcloud.com)
- [Hetzner](https://hetzner.com)
- [Scaleway](https://scaleway.com)
- [Mistral](https://mistral.ai)

Having previously used Scaleway gave it an edge, but Mistral recently launched [EU-residency options](https://mistral.ai/news/regional-inference-open-models-new-compute/), which made things even easier. Just use Mistral EU servers with ML4.

### Model

I had already chosen the model — ML4.

### Decisions

**Zed + ML4 on Mistral EU**

---

## Setup

### Zed

Zed was easy enough to set up. The Linux script worked fine for me, but there are also Windows and Mac options available on [their site](https://zed.dev/download). Just download, install, and you should be good to go.

I did also use [zed-101-setup](https://github.com/jellydn/zed-101-setup), but edited it further, so I'd say it is optional.

### Mistral

This took a bit more time. Previously one could create API keys without having an active subscription or being on a pay-as-you-go plan, but apparently they changed that on October 7th, 2026. Now, I need an API key for Zed to connect to Mistral, so I had to choose between the two options: pay-as-you-go or a Pro tier subscription. The first option requires only a billing method, the second costs €18/month. Not wanting to spend too much time on this, I just went with the latter — the Pro subscription.

Once that's active, you need to create an API key:

1. Go to [chat.mistral.ai](https://chat.mistral.ai) and log in
2. Top left, click the "Vibe" dropdown
3. Select "Admin"
4. Under API -> API Keys -> New key
5. Name it (I chose "zed") and add some future expiry date

### Connecting Zed with Mistral

1. Open Zed
2. Top-left dropdown -> Open Settings (or just `Ctrl + ,`)
3. Edit `settings.json`
4. Search for `language_models` and add the `mistral` field inside it, like so:

```json
{
  "language_models": {
    "mistral": {
      "api_url": "https://api.eu.mistral.ai/v1",
      "available_models": [
        {
          "name": "mistral-large-4",
          "display_name": "Mistral Large 4 (EU)",
          "max_tokens": 1000000,
          "supports_tools": true
        }
      ]
    }
  }
}
```

_Notice the `api_url` — this is how we select the region._

It's important to note that currently not everything is supported when using regional endpoints, see [the docs](https://docs.mistral.ai/inference/regional-inference).

Next, take a look at the [sandbox permissions](https://zed.dev/docs/ai/agent-settings#sandbox-permissions). This is how I set mine up (also in `settings.json`, at the top level):

```jsonc
// Persistent sandbox permission grants
// https://zed.dev/docs/ai/agent-settings#sandbox-permissions
"sandbox_permissions": {
  "network_hosts": [
    // Python / pip
    "pypi.org",
    "files.pythonhosted.org",

    // Node / npm
    "*.npmjs.org",
    "*.npmjs.com",

    // Elixir / Hex
    "hex.pm",
    "repo.hex.pm",

    // Rust / Cargo
    "crates.io",
    "static.crates.io",

    // GitHub (HTTPS git, API, raw)
    "github.com",
    "*.githubusercontent.com",
    "codeload.github.com",

    // Ruby
    "rubygems.org",
    "index.rubygems.org"
  ],
  "write_paths": [
    "/home/<user>/zed-sandbox"
  ]
}
```

Alright, that should be it! After saving the settings you can open a new AI agent session (for me `Ctrl + Shift + E`, but `Ctrl + Shift + P` -> `agent: focus agent` also works). Then select the model "Mistral Large 4 (EU)".

Cool. Hopefully I haven't missed something too important.

Now that that's done, the fun part can begin!

---

## Resources

- [Zed download](https://zed.dev/download) · [Zed agent settings & sandbox permissions](https://zed.dev/docs/ai/agent-settings#sandbox-permissions)
- [Mistral regional inference docs](https://docs.mistral.ai/inference/regional-inference)
- [Mistral EU-residency announcement](https://mistral.ai/news/regional-inference-open-models-new-compute/)
- [zed-101-setup](https://github.com/jellydn/zed-101-setup)

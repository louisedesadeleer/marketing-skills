# Ad Variation Generator

A [Claude Code](https://claude.com/claude-code) skill that generates ad headline variations directly in your Figma file.

Point it at a Figma file with your winning ads and it will:

1. **Read your product context** — pulls positioning, value props, target audience, and tone from whatever you give it: a messaging doc, your homepage URL, a pitch deck, or even a few paragraphs pasted into the chat.
2. **Analyze winning ads** — connects to Figma via the [ClaudeTalkToFigma](https://www.figma.com/community) plugin, finds your winner frames, and identifies the headline angle (analogy, contrast, competitive, command, provocation).
3. **Clone & populate variations** — duplicates each winner N times below the originals, swaps in new headlines that match the original angle but draw from your product's pain points and differentiators.

No copying ads into Figma by hand. No staring at a blank canvas. The variations land already laid out, named, and ready to test.

## Why this exists

Most ad iteration is bottlenecked on copywriting, not design. Once you find a winner, the fastest move is to keep its layout and test new headlines — but doing that manually in Figma is slow. This skill takes "I have 3 winning ads, generate 5 variations each with new headlines" from an afternoon to a few minutes.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- The [ClaudeTalkToFigma](https://www.figma.com/community) Figma plugin installed and running on your file
- Some form of product marketing context (a doc, a homepage URL, a pitch deck, or pasted text)
- *(Optional)* A Google Sheet with copy guidelines — needs the [Google Workspace CLI](https://www.npmjs.com/package/@anthropic-ai/googleworkspace-cli) authenticated

Full one-time setup steps for both the Figma plugin and the Google Workspace CLI are in [SKILL.md](SKILL.md).

## Install

```bash
git clone https://github.com/louisedesadeleer/marketing-skills.git ~/.claude/skills/ad-variation-generator
```

Restart Claude Code and the skill is available.

## Usage

In Claude Code, ask:

> "Create 5 ad variations of the winners in this Figma file: [URL]. Use this product doc: [link]."

The skill will ask for:
1. Your **product marketing context** (a Notion page, Google Doc, homepage URL, pitch deck, or just paragraphs in chat — whatever describes positioning, audience, and tone)
2. The **Figma file URL** with your winning ads
3. *(Optional)* Your **copy guidelines** (Google Sheet or doc with character limits, angle categories, etc.) — defaults to 25 chars max if not provided
4. **How many variations** per winner (default: 5)

It will then connect to Figma, scan for winners, and clone+populate variations in a row below each original.

## How it picks headlines

For each winning ad, the skill identifies the headline angle and keeps variations in the same family:

- **Analogy** — "Like [known tool], but for [use case]"
- **Contrast** — "[Do this], not [that]"
- **Competitive** — "[Competitor], but [advantage]"
- **Command** — "[Action verb] + [benefit]"
- **Provocation** — "Why [common behavior] when you can [better way]?"

The raw material — the pain points, differentiators, and tone — comes from your product doc. The structure stays close to whatever made the original ad work.

## License

MIT — see [LICENSE](LICENSE).

Built by [Louise de Sadeleer](https://github.com/louisedesadeleer), Growth at [Tella](https://tella.tv).

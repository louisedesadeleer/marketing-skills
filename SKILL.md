# Ad Variation Generator

Generate ad headline variations from winning ads in Figma. Reads your product marketing context, identifies winning ad frames, and creates cloned variations with new headlines directly in your Figma file.

## When to use

- User says "create ad variations," "make variations of winning ads," "generate headline variants," "clone ads with new headlines," or "test new headlines on winning ads"

## What you need from the user

Ask for these three things before starting:

1. **Product marketing context** (required) — anything that describes the product's positioning, value props, target audience, and key messaging. This is what the generated headlines will draw from. This could be:
   - A messaging doc or brand manifesto (Notion, Google Doc, markdown, PDF)
   - A link to their website or homepage (Claude will extract positioning from it)
   - A pitch deck or one-pager
   - Even just a few paragraphs pasted into the chat

   The better the input, the better the headlines — but *something* is better than nothing. Most companies have at least a homepage.
2. **Figma file URL** (required) — the Figma file containing the winning ad frames to use as templates.
3. **Copy guidelines** (optional) — a Google Sheet, doc, or list with headline examples, character limits, or copy rules. If not provided, default to a **max 25 characters** per headline.

Also ask:
- **Number of variations** per winner (default: 5)
- Whether they've completed the one-time setup below (Figma plugin + Google Sheets CLI)

## One-time setup

Before running this skill for the first time, the user needs two things connected: the Figma MCP plugin and (if using Google Sheets for copy guidelines) the Google Workspace CLI.

### 1. Google Workspace CLI setup

This lets Claude create spreadsheets, read headline data, and write rows from the terminal. Skip this section if you're providing copy guidelines as plain text or a doc.

#### Step 1: Install the Google Workspace CLI

```bash
npm install -g @anthropic-ai/googleworkspace-cli
```

Or use it via npx (no global install needed):
```bash
npx @googleworkspace/cli
```

> **Tip:** Be careful with spaces in commands — the CLI can be picky about formatting.

#### Step 2: Install the Google Cloud CLI

The workspace CLI needs Google Cloud authentication to talk to your Google account.

```bash
brew install --cask google-cloud-sdk
```

This takes ~2 minutes to install. Once done, authenticate:

```bash
gcloud auth login
```

This opens a browser window. Sign in with the Google account that owns your Sheets.

#### Step 3: Run GWS auth setup

```bash
gws auth setup
```

This walks you through an interactive setup:
1. **Select your Google account** — press Enter to confirm
2. **Create or select a GCP project** — if you don't have one, select "Create a new project" and name it anything (e.g., "marketing-tools")
3. **Select APIs to enable** — use arrow keys to select **Google Sheets**, then press Enter

#### Step 4: Configure the OAuth consent screen

The CLI will give you a URL to visit. When you land on the Google Cloud Console:

1. If you can't see your project, click the project dropdown at the top and look under "All" or your organization
2. Give your app a name (e.g., "Sheets to Figma Ads")
3. Add your email as the support email
4. Select **External** for the audience type
5. Add your email as a contact, agree, and click **Create**

#### Step 5: Create OAuth credentials

Back in the terminal, the CLI prompts you to create an OAuth client:

1. Go to the credentials page in Google Cloud Console (the CLI gives you the URL)
2. Click **+ Create Credentials** → **OAuth client ID**
3. Application type: **Desktop app**
4. Name it anything, click **Create**
5. Copy the **Client ID** and **Client Secret** — paste them into the terminal when prompted

> ⚠️ These are secret credentials — don't share them publicly.

#### Step 6: Authenticate GWS

```bash
gws auth login
```

1. The CLI asks you to confirm the auth scopes — press Enter
2. It gives you a URL to open in your browser
3. Allow access when prompted
4. Close the browser tab when it says "success"

#### Step 7: Enable the Google Sheets API

```bash
gcloud services enable sheets.googleapis.com --project=YOUR_PROJECT_NAME
```

Replace `YOUR_PROJECT_NAME` with the project name you created (e.g., `marketing-tools`).

#### Step 8: Verify it works

Create a test spreadsheet to confirm everything is connected:

```bash
gws sheets spreadsheets create --json '{"properties": {"title": "Test - Delete Me"}}'
```

You should get back a spreadsheet URL. Open it to confirm, then delete it.

> **Pro tip:** If you use Claude Code, you can now just say "create a Google spreadsheet called X" and Claude will use the CLI for you — no need to memorize these commands.

---

### 2. Figma MCP plugin: Claude Talk to Figma

This plugin lets Claude read and manipulate Figma files in real time through a WebSocket connection.

#### Step 1: Install the MCP package

```bash
npx claude-talk-to-figma-mcp
```

This downloads the documentation and packages needed for the MCP.

#### Step 2: Import the Figma plugin

1. Open Figma
2. Go to **Plugins** (bottom of the left panel or via the menu)
3. Click **Import plugin from manifest...**
4. Navigate to the downloaded `claude-talk-to-figma-mcp` folder → go into `src/` → select `manifest.json`

> **Important:** Make sure you select the `manifest.json` inside the `src/` folder, not the root of the project.

#### Step 3: Configure the MCP for Claude Code

Copy the MCP configuration from the plugin's README and tell Claude Code to install it:

```
Install this MCP: { "mcpServers": { "claude-talk-to-figma": { "command": "npx", "args": ["claude-talk-to-figma-mcp"] } } }
```

Restart Claude Code, then check your MCP list to confirm "claude-talk-to-figma" appears and is connected.

#### Step 4: Connect to Figma

1. Open your Figma file with the ad designs
2. Go to **Plugins** → **Claude Talk to Figma** → **Run**
3. The plugin panel shows a **channel name** (e.g., `abc123`)
4. Copy this channel name and give it to Claude so it can connect

> **Keep Figma open** with the plugin running throughout the session — the connection is live via WebSocket.

---

## Workflow

### Step 1: Read product marketing context

Read the product marketing context the user provided. If they gave a website URL, fetch and analyze the homepage, about page, and any product/features pages to extract positioning. If they gave a doc, read it directly. Extract:
- Core value proposition and positioning
- Target audience and their pain points
- Key differentiators vs competitors
- Existing taglines, one-liners, or messaging frameworks
- Tone of voice (casual, professional, provocative, etc.)

This context drives all headline generation. Keep it loaded throughout the session.

### Step 2: Read copy guidelines (if provided)

If the user provided a Google Sheet or doc with copy guidelines:
- Extract the spreadsheet ID from the URL (the part between `/d/` and `/edit`)
- Fetch all data with an open-ended range:
  ```bash
  gws sheets spreadsheets values get --params '{"spreadsheetId": "<ID>", "range": "A:Z"}'
  ```
- Identify: headline examples, character limits per format, angle categories, any naming conventions

If no copy guidelines were provided, use these defaults:
- Max headline length: **25 characters**
- No specific angle constraints — derive angles from the product marketing doc

### Step 3: Generate headlines into a Google Sheet

Before touching Figma, generate all headlines into a Google Sheet so the user can review and edit them:

1. Create a new spreadsheet:
   ```bash
   gws sheets spreadsheets create --json '{"properties": {"title": "Ad Headlines - [Month Year]"}}'
   ```
2. Populate it with columns: **Angle**, **Headline**, **Character Count**, **Winner Reference**
3. Generate headlines drawing from the product marketing doc, respecting character limits
4. Share the spreadsheet URL with the user for review

> **Tip:** When using the CLI from Claude Code, special characters in headline text can cause shell escaping issues. If `gws` append commands fail, try writing headlines one at a time or use simpler punctuation.

### Step 4: Connect to Figma & analyze winners

1. Join the Figma channel via `mcp__ClaudeTalkToFigma__join_channel`
2. Get document info via `mcp__ClaudeTalkToFigma__get_document_info` to find all winner frames
3. Get detailed node info for each winner via `mcp__ClaudeTalkToFigma__get_nodes_info`

For each winning ad frame, identify:
- **The headline text node** (largest font size, typically the main message)
- **The angle/style** of the headline (analogy, contrast, competitive, pain point, benefit, etc.)
- **The CTA text** and other supporting text nodes
- **Frame dimensions and position** for layout planning

### Step 5: Clone and populate in Figma

1. **Clone each winner frame** N times using `mcp__ClaudeTalkToFigma__clone_node`, positioning them in a row below the originals:
   - Row spacing: original frame height + 120px gap
   - Column spacing: frame width + 40px gap
   - Each winner's variations get their own row

2. **Find headline text nodes in every clone** using `mcp__ClaudeTalkToFigma__scan_text_nodes` on each clone individually. Figma node IDs are globally incremented, not per-frame, so you cannot predict IDs from one clone to another. Always scan each clone for its own text nodes.

3. **Update headline text** in each clone via `mcp__ClaudeTalkToFigma__set_text_content`

4. **Rename each clone** via `mcp__ClaudeTalkToFigma__rename_node` using format: `W[winner#] — V[variation#]`

> **Important:** When using the Figma MCP WebSocket commands, parameters use camelCase — e.g., `nodeId` not `node_id`. Getting this wrong will silently fail.

### Step 6: Summarize

Present a clean summary table:
- Winner name → original headline → angle style
- Each variation with its new headline and character count
- Note the layout in Figma (rows below originals)

## Tips for great variations

- **Don't just rephrase** — shift the frame. "Send videos, not emails" → "One video beats ten emails" (same angle, different frame)
- **Short > long** — the best ad headlines are scannable in under 2 seconds
- **Mix emotional and rational** — some variations should hit feelings ("Stop dreading edit day"), others hit logic ("3 min vs 3 hours")
- **Test specificity** — some variations should be specific ("Save 3hrs per video"), others broad ("Make videos effortlessly")

## Parallel execution tips

- Clone all frames for one winner in parallel (batch of 5 clone calls)
- Scan text nodes for all clones in parallel (one `scan_text_nodes` call per clone)
- Set text + rename in parallel for each clone (2 calls per clone)

## Troubleshooting

### Google Workspace CLI
- **"Unable to parse range"** — the tab might not be called "Sheet1". Query actual tab names first with `gws sheets spreadsheets get`
- **403 error on Sheets API** — the API isn't enabled. Run `gcloud services enable sheets.googleapis.com --project=YOUR_PROJECT`
- **Shell escaping errors** — special characters (quotes, ampersands) in headline text can break zsh. Simplify punctuation or write one row at a time
- **Wrong Google account** — if you have multiple Google accounts, make sure you're authenticated with the one that owns the sheets. The most annoying part of this workflow is selecting the right account every time

### Figma MCP
- **Plugin not found** — make sure you imported `manifest.json` from the `src/` folder, not the project root
- **WebSocket commands silently fail** — check that you're using `nodeId` (camelCase), not `node_id`
- **Can't connect to channel** — make sure the Figma plugin panel is open and showing the channel name. If it disconnected, close and re-run the plugin
- **Text not updating** — scan text nodes on the specific clone first. Don't assume node IDs from one clone match another

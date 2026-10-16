# Parsly for Claude Code

Study any YouTube channel from Claude Code. Parsly finds the videos that broke out, reads how they open, and turns what worked into hooks and video ideas for your own channel. The skills are free; Parsly's Chrome extension fetches the videos through your own browser, because YouTube doesn't give transcripts to servers.

## What it does for you

1. **Install Parsly in Chrome** from [parsly.to](https://parsly.to) and sign in. In its first screen, copy the command for Claude Code.
2. **Install the plugin** in Claude Code:
   ```
   /plugin marketplace add itsmattblu/parsly
   /plugin install parsly@parsly
   ```
   It adds three skills and Parsly's connection. Type `/mcp`, choose **parsly** and sign in with the same Parsly account.
3. **Ask:**
   - **Teardown** (`/parsly:teardown`): "Why did @mkbhd's last breakout work?" Finds the videos that beat the channel's usual views, reads their openings, and tells you what they share and what to try.
   - **Hooks** (`/parsly:hooks`): "How do the top sourdough channels open their videos?" A library of the openings that work in a niche, sorted by the move they make, with templates.
   - **Video ideas** (`/parsly:video-ideas`): "What should I make next for @mychannel?" Ten ideas, each built on a real breakout in your niche, with a hook for the first ten seconds.

You don't need to name a skill: ask in your own words and Claude picks the right one.

## How it does it

- **Parsly's tools** (`.mcp.json`, Parsly's server at `https://api.parsly.to/mcp`): `parsly_status` (is Parsly in Chrome connected?), `youtube_channel_videos` (a channel's uploads, read live), `youtube_read_transcripts` (up to 5 transcripts into the conversation) and `youtube_download_transcripts` (many, saved to your Downloads as a dated folder: `parsly/2026-10-15-starter-story/` with one Markdown file per video and an `index.json`), followed by `job_status`. Transcripts are fetched by Parsly in your Chrome while it's open; nothing about the videos or transcripts is kept on Parsly's server.
- **Every skill checks `parsly_status` first.** Until Parsly in Chrome is signed in and connected, the tools refuse and say how to set it up, and the skill passes that on.
- **Breakouts** are worked out by Parsly's server: `youtube_channel_videos` returns each video's views as a number and `xUsual`, its views against its neighbours' (the uploads around it), so a channel that grew still shows its real breakouts, sorted newest or biggest-breakout first, at most `limit` (60 by default). Views are YouTube's rounded counts and dates are relative, so the numbers are approximate; videos under two weeks old aren't scored.
- **Openings by default:** `youtube_read_transcripts` returns each video's first minute unless asked for `part: "full"`, which keeps the conversation small and is what the skills mostly need.
- **Big jobs work from files:** a skill downloads, moves the folder into a `parsly/` folder in Claude Code's working folder (`mkdir -p parsly && mv …`), and reads only what it needs (the index, the first lines of each file, `grep`). Next time, it uses the newest `parsly/*-<channel>/` folder instead of fetching again. Run Claude Code in a folder you keep for research, so they collect in one place.
- **The skills** are `skills/<name>/SKILL.md`: what to fetch, what to look for, and the shape of the report.
- **Not covered:** YouTube Shorts and live streams (other tabs of a channel), and searching YouTube by topic (the skills work from channels; ask Claude to suggest some).

**Where it lives:** built in mattblu's private monorepo (`products/parsly/plugin`) and published to [github.com/itsmattblu/parsly](https://github.com/itsmattblu/parsly) when it changes on `main`, as one commit by GitHub's bot (`.github/workflows/publish-plugins.yml`); edits made in the public repo are overwritten. To try a change before it's published: `claude --plugin-dir products/parsly/plugin`. Check it with `claude plugin validate --strict products/parsly/plugin`. The publish refuses if personal details are in the folder: `DRY_RUN=1 bash .github/scripts/publish-plugin.sh parsly` runs the same check.

**When it breaks.**
- *"Parsly isn't set up in this person's Chrome yet":* install the extension, sign in with the same account Claude Code signed in with, allow YouTube, keep Chrome open.
- *Two Parslys in Claude Code* (this plugin's and one added by hand, such as `parsly-local` for development): Claude may call the wrong one. Remove or disable one in `/mcp`.
- *Breakout numbers look wrong:* YouTube may have changed how it writes views or dates; `@parsly/youtube`'s `breakouts.ts` reads them.

## Licence

MIT. Parsly's server and Chrome extension aren't part of this plugin.

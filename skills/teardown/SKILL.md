---
name: teardown
description: Tear down a YouTube channel to find which videos broke out (far beyond the channel's usual views) and why, by reading their titles and the opening of their transcripts with Parsly. Use whenever someone asks why a channel or video blew up, what's working for a YouTuber, what a creator's best videos have in common, how a channel grew, or wants to "study", "break down", "analyse" or "reverse-engineer" a YouTube channel, even if they don't say "teardown". Needs the Parsly connection (its parsly_status and youtube_* tools).
---

# YouTube channel teardown

Find the videos that broke out on a channel, then work out what they share that the channel's other videos don't. The person wants something they can act on for their own channel, not a list of stats.

Parsly gets the data: it lists a channel's uploads (titles, rounded views, relative dates, lengths) and fetches transcripts through the person's own Chrome, because YouTube doesn't give transcripts to servers.

## 1. Check Parsly is ready

Call Parsly's `parsly_status` tool first. If `ready` is false, show the person its `setup` text in plain words and stop: every other Parsly tool will refuse until Parsly in Chrome is signed in and connected, so there's nothing useful to do yet. Once they've done it, check again.

## 2. Find the breakouts

Call `youtube_channel_videos` with the channel (an `@handle`, a channel URL or ID), `sort: "breakouts"` and `limit: 15`. Parsly works out the numbers: each video's `views`, and `xUsual`, how many times its neighbouring uploads' views it got (so a channel that grew still shows its real breakouts), plus the channel's `usualViews`. Videos under two weeks old have no score yet. Uploads only: Shorts and live streams aren't included, so say so if the person asks about them.

A breakout is roughly 3x or more. If nothing reaches 3x, say the channel is steady and use the top few anyway. For contrast, call it again with `sort: "newest"` and `limit: 30`, and pick 2 typical videos (`xUsual` near 1).

## 3. Read what made them work

Read the openings of the top 3 to 5 breakouts, plus the 2 typical videos for contrast, with `youtube_read_transcripts` (up to 5 videos per call; make two calls). It returns just the first minute by default, which is where videos win or lose; ask for `part: "full"` only for a video whose structure you need. It waits up to about 45 seconds; if it answers with a job ID instead, call `job_status` with it until it's done. If it says Chrome stopped checking in, tell the person to open Chrome and try again.

For each video, look at:

- **The title:** the promise, the specific number or name, the curiosity gap, the stakes.
- **The opening** (the first minute): how it opens, what it promises, how fast it gets there.
- **The topic and the angle,** compared with the channel's usual ones.
- **The length.**

Then compare: what do the breakouts share that the typical videos don't? One pattern backed by three videos is worth more than five observations about one video.

## 4. Report

Use this shape, short and concrete:

```markdown
# <Channel>: what broke out and why

**Usual video:** ~<views>. **Breakouts:** <n> videos at 3x or more.

| Video | Views | x usual | Age |
|---|---|---|---|
| [Title](link) | 1.2M | 9.1x | 4 months |

## What the breakouts share
- **<Pattern>:** <what it is, with a short quote from an opening or title as evidence, and which videos>.
(2 to 4 patterns)

## What the typical videos do instead
<One or two lines, so the contrast is clear.>

## Try this next
- <3 to 5 specific moves for the person's own channel: a title shape, an opening line to adapt, a topic angle.>

<One line on limits: views are rounded, dates relative, Shorts not included.>
```

Quote openings briefly (a sentence or two, attributed to the video). The person is studying them, not republishing them.

## Working from files (big jobs)

For more than a handful of videos (a whole channel, or 20 or more across channels), work from files instead of reading transcripts into the conversation:

1. **Look for an earlier download first.** Downloads live in a `parsly/` folder in the current working folder, one folder per download, dated and kebab-case: `parsly/2026-10-15-starter-story/` holds `index.json` (each video's file, link, views, `xUsual`, length) and one Markdown file per video (`001-video-title.md`, newest first, transcript with times). If there's a recent one for this channel (the newest `parsly/*-<channel-in-kebab-case>/`), use it, unless the person wants fresh numbers.
2. **Otherwise download:** `youtube_download_transcripts` (a channel, or video IDs). Parsly saves a new dated folder to the person's Downloads; follow it with `job_status` until done. Its summary ends with the folder's full path.
3. **Move it into the working folder** so everything stays together: `mkdir -p parsly && mv "<full path>" parsly/`.
4. **Read only what you need:** the index for the numbers, `head` for openings (the first ~15 lines of a file is roughly the first minute), `grep` across files for a phrase, and whole files only for the few videos that matter.

## When something goes wrong

- **Not a channel:** the tool says so; ask for the `@handle` or channel URL.
- **No captions:** some videos have none; say which, and judge those on title and topic only.
- **Many videos wanted** (the whole channel, or more than 10): work from files (above).

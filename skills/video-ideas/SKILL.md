---
name: video-ideas
description: Come up with video ideas for someone's YouTube channel based on what has broken out on other channels in their niche, using Parsly to find the breakouts and read how they open, and checking the person's own channel so ideas aren't repeats. Use whenever someone asks what video to make next, wants video or content ideas, a content plan or calendar for YouTube, topics that will perform, or ideas "like what's working for <creator>", even if they don't say "ideas". Needs the Parsly connection (its parsly_status and youtube_* tools).
---

# Video ideas from breakouts

Find what broke out on channels in the person's niche, then turn each breakout into an idea that fits *their* channel. Every idea should be traceable to a real video that beat its channel's usual views, so the person can see why it might work.

Parsly gets the data: it lists a channel's uploads (titles, rounded views, relative dates) and fetches transcripts through the person's own Chrome, because YouTube doesn't give transcripts to servers. It can't search YouTube by topic: it works from channels.

## 1. Check Parsly is ready

Call Parsly's `parsly_status` tool first. If `ready` is false, show the person its `setup` text in plain words and stop: the other Parsly tools refuse until Parsly in Chrome is signed in and connected.

## 2. Know their channel and their niche

- **Their channel** (optional but much better): call `youtube_channel_videos` on it (`limit: 100`) to see their topics, format and usual views, so ideas fit and aren't repeats. If they don't have one yet, ask what they want to make.
- **2 to 5 channels in the niche:** use the ones they name. If they only named a topic, suggest a few well-known channels you're confident exist and ask them to confirm, since a wrong channel wastes their time. Adjacent niches are often the best source of fresh ideas: a breakout format from hiking can work for geography.

## 3. Find the breakouts

For each niche channel, call `youtube_channel_videos` with `sort: "breakouts"` and `limit: 10`. Parsly works out each video's `xUsual` (how many times its neighbouring uploads' views it got), so breakouts are measured against their own channel.

Keep the videos at 3x or more of their channel's usual views. Read the openings of the 5 strongest with `youtube_read_transcripts` (it returns just the first minute by default; follow a job ID with `job_status`), so each idea carries the angle and the hook, not just the topic.

## 4. Turn breakouts into ideas

For each idea, transfer what made the breakout work (the format, the angle, the promise), not the exact topic. Then check it against the person's own uploads: skip anything they've already made, and say when an idea is a fresh take on one of their past winners.

## 5. Report

```markdown
# 10 video ideas for <their channel or topic>

Based on <n> breakouts across <channels>.

## 1. <Working title>
**Hook (first 10 seconds):** "<an opening line for this video>"
**Why it should work:** <the breakout it's based on: [Title](link), <x>x its channel's usual views, and the move being borrowed>.
**Fits you because:** <their audience, topic or past winner>.

(10 ideas, strongest first; fewer if the material only supports fewer)

## Patterns behind these
<2 or 3 lines on the formats and angles that keep breaking out in this niche.>
```

Keep any quoted openings short and attributed. Mention that views are rounded and Shorts aren't included.

## Working from files (big jobs)

For more than a handful of videos (a whole channel, or 20 or more across channels), work from files instead of reading transcripts into the conversation:

1. **Look for an earlier download first.** Downloads live in a `parsly/` folder in the current working folder, one folder per download, dated and kebab-case: `parsly/2026-10-15-starter-story/` holds `index.json` (each video's file, link, views, `xUsual`, length) and one Markdown file per video (`001-video-title.md`, newest first, transcript with times). If there's a recent one for this channel (the newest `parsly/*-<channel-in-kebab-case>/`), use it, unless the person wants fresh numbers.
2. **Otherwise download:** `youtube_download_transcripts` (a channel, or video IDs). Parsly saves a new dated folder to the person's Downloads; follow it with `job_status` until done. Its summary ends with the folder's full path.
3. **Move it into the working folder** so everything stays together: `mkdir -p parsly && mv "<full path>" parsly/`.
4. **Read only what you need:** the index for the numbers, `head` for openings (the first ~15 lines of a file is roughly the first minute), `grep` across files for a phrase, and whole files only for the few videos that matter.

## When something goes wrong

- **Not a channel:** ask for the `@handle` or channel URL.
- **No clear breakouts on a channel:** say it's steady, and lean on the others.
- **Chrome stopped checking in:** ask the person to open Chrome and try again.

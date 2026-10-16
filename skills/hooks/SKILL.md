---
name: hooks
description: Build a library of the opening lines (hooks) that work in a YouTube niche, by reading the first 30 to 60 seconds of the best-performing videos on a few channels with Parsly and sorting them by the move they make. Use whenever someone wants better video intros or openings, asks how top creators start their videos, wants hook ideas, hook templates or "the first 10 seconds" for their niche, or wants to study what keeps viewers watching, even if they don't say "hooks". Needs the Parsly connection (its parsly_status and youtube_* tools).
---

# Hook library

Collect how the best-performing videos in a niche open, sort them by the move they make, and turn that into openings the person can use. The goal is a short, usable library, not a transcript dump.

Parsly gets the data: it lists a channel's uploads (titles, rounded views, relative dates) and fetches transcripts through the person's own Chrome, because YouTube doesn't give transcripts to servers. It can't search YouTube by topic: it works from channels.

## 1. Check Parsly is ready

Call Parsly's `parsly_status` tool first. If `ready` is false, show the person its `setup` text in plain words and stop: the other Parsly tools refuse until Parsly in Chrome is signed in and connected.

## 2. Pick the channels and the videos

You need 2 to 5 channels in the niche. If the person named some, use them. If they only named a topic, suggest a few well-known channels you're confident exist and ask them to confirm or swap them, since a wrong channel wastes their time.

For each channel, call `youtube_channel_videos` with `sort: "breakouts"` and `limit: 10`. Parsly works out each video's `xUsual` (how many times its neighbouring uploads' views it got), so breakouts are measured against their own channel.

Take each channel's top 5 to 8 by "x usual", so the hooks come from videos that beat their own channel, not just from the biggest channels. Aim for 15 to 30 videos in all.

## 3. Read the openings

- **Up to 5 videos at a time:** `youtube_read_transcripts`, which returns just each video's opening (the first minute) by default, exactly what a hook library needs; follow a job ID with `job_status`. Several calls are fine.
- **Many (20 or more):** work from files (below): download them, keep them in `parsly/`, and read only each file's first minute.

From each, take the opening: the words before the video settles into its main content, usually the first 30 to 60 seconds. Note the title too, since the hook usually pays off the title's promise.

## 4. Sort them by move

Group the openings by what they do. Common moves (use your own if the material suggests better ones):

- **Result first:** shows the outcome, then how ("This loaf cost 40p and beat a £6 bakery one").
- **The mistake:** names an error the viewer is probably making.
- **The challenge or dare:** a constraint and a time ("100 days", "only old maps").
- **The question:** a question the viewer now needs answered.
- **Cold open on the story:** starts mid-scene, explains later.
- **Contrarian:** says the common advice is wrong.
- **Stakes:** money, time or risk, stated straight away.

## 5. Report

```markdown
# Hooks that work in <niche>

From <n> top-performing videos across <channels>.

## <Move> (<k> videos, typically <x>x usual)
- "<opening line, a sentence or two>" (<Title>, <channel>, <views>, <x>x)
**Template:** <a fill-in-the-blanks version>

(one section per move, the strongest first)

## What the best ones have in common
<2 or 3 lines: speed to the promise, specificity, what they leave out.>

## Openings for you
<3 to 5 openings written for the person's own topic, if they gave one, each using a move above.>
```

Quote openings briefly, attributed to the video: the person is studying them, not republishing them. Mention that views are rounded and Shorts aren't included.

## Working from files (big jobs)

For more than a handful of videos (a whole channel, or 20 or more across channels), work from files instead of reading transcripts into the conversation:

1. **Look for an earlier download first.** Downloads live in a `parsly/` folder in the current working folder, one folder per download, dated and kebab-case: `parsly/2026-10-15-starter-story/` holds `index.json` (each video's file, link, views, `xUsual`, length) and one Markdown file per video (`001-video-title.md`, newest first, transcript with times). If there's a recent one for this channel (the newest `parsly/*-<channel-in-kebab-case>/`), use it, unless the person wants fresh numbers.
2. **Otherwise download:** `youtube_download_transcripts` (a channel, or video IDs). Parsly saves a new dated folder to the person's Downloads; follow it with `job_status` until done. Its summary ends with the folder's full path.
3. **Move it into the working folder** so everything stays together: `mkdir -p parsly && mv "<full path>" parsly/`.
4. **Read only what you need:** the index for the numbers, `head` for openings (the first ~15 lines of a file is roughly the first minute), `grep` across files for a phrase, and whole files only for the few videos that matter.

## When something goes wrong

- **Not a channel:** ask for the `@handle` or channel URL.
- **No captions:** skip the video and say so.
- **Chrome stopped checking in:** ask the person to open Chrome; downloads wait up to a day.

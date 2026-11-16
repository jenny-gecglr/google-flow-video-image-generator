---
name: google-flow-video-generator
description: Drives Google Flow in a logged-in Chrome session to generate images and videos from a prompt file, name every output with a prompt number, download them, check for missing items, build a storyboard and a rough-cut preview. Use when someone says "generate this in Flow", "run this prompt file", "make the scenes", or "storyboard".
---

# Google Flow Video Generator

Flow has no official API, so this skill drives the Flow web app through browser automation (for example Claude in Chrome) using the user's own logged-in session. Work one step at a time, at human speed.

> Check Google's terms of use and accept the account risk before automating Flow.

## Non-negotiable rules

1. **Ask before spending credits.** Any action that costs credits gets one short question: "This costs X credits. Approve?" Do not press Approve before a yes. **Never choose "always approve in project."** Free actions (for example image models at 0 credits) can start without asking.
2. **Target: images at 4K, video at 1080p. Never upscale video to 4K** (very expensive in credits). Upscale images outside Flow (see Download below). Do not use Flow's own bulk 4K download option, because its quality is lower; the project zip described below is fine.
3. **One Flow project per video.** Old or heavy projects freeze the browser tab.
4. **Always show the real cost.** Credit prices change by model and resolution. Read the price Flow displays and quote it; do not rely on remembered numbers.
5. **Number every output after generating** (see below). Syncing with the script depends on it.
6. **Never delete or move files without asking.** Browsers add `(1)` copies on repeated downloads: list them, get a yes, then delete.

## Input

A prompt file produced from the video script. Each prompt has a label of the form `NNN | IMAGE or VIDEO | start-end | duration` (for example `001 | IMAGE | 00:00.0-00:06.0 | 6.0 s`), followed by a single paragraph. Prompts come in groups of 24 with blank lines between groups. A trailing `CHECK` line (a closing note, not a scene) is not a prompt. For the rough cut you also need the narration audio track and, optionally, its subtitles.

If the file starts with a shared "world vocabulary" block (the characters, symbols and visual style used by every scene), paste that block before **every** group. Flow does not remember it between submissions.

## Workflow

1. **Generate.** Open `flow.google.com`, create a new project. Put the vocabulary block plus one 24-prompt group on the clipboard (verify the clipboard content) and paste with Ctrl+V into the agent box (do not type long text). Prefix: "For each block, generate an IMAGE (16:9) or a silent VIDEO (6 seconds by default) by its label, tell me the credit cost, and wait for my approval." Show the cost to the user, wait for a yes, then press only Approve (the plain button, never "always approve"). If the displayed cost is 0 credits, skip the question and start.
2. **Rename.** Ask the agent to rename all items with their prompt number (001, 002, ...). It handles at most 24 items at a time, so use "Rename first 24" and "Rename remaining" in turn, then verify the list.
3. **Download.**
   - *Main route:* project menu, then download the project zip. It contains images at 1K and video at 720p. Upscale the images to 4K outside Flow (for example Real-ESRGAN 4x on a free cloud GPU notebook) and bring the videos to 1080p with `ffmpeg`. Keep the prompt number in every file name after upscaling.
   - *Backup route:* an optional Chrome extension that downloads the project's items, which the user starts with "download now" or schedules overnight. Only this route can deliver larger sizes directly; if a size is not available it falls back from 4K to 2K to 1K for images and from 1080p to 720p for video. Without such an extension, use the main route.
   - *Cautions:* do not download files by hand in the browser. Do not download in parallel: Flow shows reCAPTCHA and "one upscale at a time" warnings. Files typically arrive in the Downloads folder as `NNN_hash.ext`.
4. **Check for gaps.** Count the numbered files and report missing numbers.
5. **Storyboard.** Make 640 px previews with `ffmpeg`, label each with number, type and time, and compose them into sheets of 24. Missing items appear as gray boxes. Keep prompt order. Display the finished sheets to the user (open or attach the images).
6. **Rough cut.** Scaffold a HyperFrames project (HyperFrames is an HTML-to-video tool, run with `npx hyperframes`) and:
   - take durations from the prompt labels;
   - use lightweight 1920 px preview copies (4K freezes low-memory machines);
   - cut audio from the narration track with the same time ranges and check it against the subtitle timings;
   - add a gentle Ken Burns move with 0.25 s fade in and out, and keep video clips muted;
   - run `npx hyperframes check --json`, then preview.
   Switch to full-resolution files only for the final render.

## Single image requests

Prerequisites: Chrome is open, the user is logged in to Flow, and the browser automation is connected. For "generate this one image": open a project, give the prompt to the agent, start it if it costs 0 credits, download the image when done (project menu or the optional extension), upscale it to 4K outside Flow if needed, and show the result. It is not instant: about 10-20 s to generate, plus upscaling time.

## Lessons learned

- The Flow agent handles 24 items per run, and the bottom strip shows only that group.
- Local upscaling on a weak laptop (plain `ffmpeg` scaling, or Real-ESRGAN on a CPU or integrated GPU) gave poor or unusable results; use a free cloud GPU notebook instead.
- A separate automated browser (Playwright) never finished Flow's own upscale step; the real Chrome profile worked.
- Browser tabs freeze with lots of media. If screenshots time out, read the page text or use JavaScript, then reload.

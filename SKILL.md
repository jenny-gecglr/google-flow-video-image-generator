---
name: google-flow-video-generator
description: Drives Google Flow in a logged-in Chrome session to generate images and videos from a prompt file, name every output with a prompt number, download them, check for missing items, build a storyboard and a rough-cut preview. Use when someone says "generate this in Flow", "run this prompt file", "make the scenes", or "storyboard".
---

# Google Flow Video Generator

Flow has no official API, so this skill drives the Flow web app through browser automation (for example Claude in Chrome) using the user's own logged-in session. Work one step at a time, at human speed.

> Flow is not driven through an official integration. Check Google's terms of use and accept the account risk before automating it.

## Non-negotiable rules

1. **Ask before spending credits.** Any action that costs credits gets one short question: "This costs X credits. Approve?" Do not press Approve before a yes. **Never choose "always approve in project."** Free actions (for example image models at 0 credits) can start without asking.
2. **Images at 4K, video at 1080p. Never upscale video to 4K** (very expensive in credits). If you upscale images, do it outside Flow (for example Real-ESRGAN on a free GPU notebook); Do not use Flow's own bulk 4K download; its quality is lower. Local upscaling on a weak laptop (plain `ffmpeg` scaling, CPU/integrated-GPU Real-ESRGAN) gave poor or unusable results, so use a free cloud GPU notebook instead.
3. **One Flow project per video.** Old or heavy projects freeze the browser tab.
4. **Always show the real cost.** Credit prices change by model and resolution. Read the price Flow displays and quote it; do not rely on remembered numbers.
5. **Number every output after generating** (see below). Syncing with the script depends on it.
6. **Never delete or move files without asking.** Browsers add `(1)` copies on repeated downloads: list them, get a yes, then delete.

## Input

A prompt file produced from the video script. Format: a label `NNN | IMAGE|VIDEO | start-end | duration`, followed by a single paragraph. Prompts come in groups of 24 with blank lines between groups. A trailing `CHECK` line is not a prompt.

If the file starts with a shared "world vocabulary" block (characters, symbols, style), paste that block before **every** group. Flow does not remember it between submissions.

## Workflow

1. **Generate.** Open `flow.google.com`, create a new project. Put the vocabulary block plus one 24-prompt group on the clipboard (verify the clipboard content) and paste with Ctrl+V into the agent box (do not type long text). Prefix: "For each block, generate an IMAGE (16:9) or a silent 6-second VIDEO by its label, tell me the credit cost, and wait for my approval." Show the cost to the user, wait for a yes, then press only Approve.
2. **Rename.** Ask the agent to rename all items with their prompt number (001, 002, ...). It handles at most 24 items at a time, so use "Rename first 24" and "Rename remaining" in turn, then verify the list.
3. **Download.** Main route: project menu, then download the project zip (images come at 1K, video at 720p); upscale images to 4K outside Flow (for example Real-ESRGAN 4x on a free GPU notebook) and bring videos to 1080p with `ffmpeg`. Backup route: a download helper extension (the user runs it, with "download now" or an overnight schedule). If a size is not available, fall back from 4K to 2K to 1K for images (1080p to 720p for video). Do not download files by hand in the browser; use the project menu or the helper extension. Files land in the Downloads folder as `NNN_hash.ext`.
4. **Check for gaps.** Count the numbered files and report missing numbers. Do not download in parallel; Flow shows reCAPTCHA and "one upscale at a time" warnings.
5. **Storyboard.** Show the finished sheets to the user in your interface. Make 640 px previews with `ffmpeg`, label each with number, type and time, and compose them into sheets of 24. Missing items appear as gray boxes. Keep prompt order.
6. **Rough cut.** Scaffold a HyperFrames project. Take durations from the prompt labels, use lightweight 1920 px preview copies (4K freezes low-memory machines), cut audio from the narration track with the same time ranges, verify against the subtitles, add a gentle Ken Burns move with 0.25 s fade in and out, and keep video clips muted. Run `npx hyperframes check --json`, then preview. Switch to full-resolution files only for the final render.

## Single image requests

Prerequisites: Chrome is open, the user is logged in to Flow, and the browser automation is connected. For "generate this one image": open a project, give the prompt to the agent, start it if it is free, download when done, and show the result. It is not instant: about 10-20 s to generate, plus upscaling time.

## Lessons learned

- The Flow agent handles 24 items per run, and the bottom strip shows only that group.
- A separate automated browser (Playwright) never finished the upscale; the real Chrome profile worked.
- Browser tabs freeze with lots of media. If screenshots time out, read the page text or use JavaScript, then reload.

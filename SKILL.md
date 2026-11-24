---
name: google-flow-video-image-generator
description: Drives Google Flow in a logged-in Chrome session to generate images and videos from a prompt file, name every output with a prompt number, download them, check for missing items, build a storyboard and a rough-cut preview. Use when someone says "generate this in Flow", "run this prompt file", "make the scenes", or "storyboard".
---

# Google Flow Video and Image Generator

Flow has no official API, so this skill drives the Flow web app through browser automation (for example Claude in Chrome) using the user's own logged-in session. Work one step at a time, at human speed.

> Check Google's terms of use and accept the account risk before automating Flow.

## Non-negotiable rules

1. **Ask before spending credits.** Any action that costs credits gets one short question: "This costs X credits. Approve?" Do not press Approve before a yes. **Never choose "always approve in project."** Free actions (for example image models at 0 credits) can start without asking.
2. **Target: images at 4K, video at 1080p. Never upscale video to 4K** (very expensive in credits). Upscale images outside Flow (see Download below). Do not use Flow's own bulk 4K download option, because its quality is lower; the project zip described below is fine.
3. **One Flow project per video.** Old or heavy projects freeze the browser tab.
4. **Always show the real cost.** Credit prices change by model and resolution. Read the price Flow displays and quote it; do not rely on remembered numbers. Reference points at the time of writing: a 6 second 720p video cost 18 credits, and a 4K video about 50, which is why video is never upscaled to 4K. When you see a new model or resolution, ask.
5. **Number every output after generating** (see below). Syncing with the script depends on it.
6. **Never delete or move files without asking.** Browsers add `(1)` copies on repeated downloads: list them, get a yes, then delete.

## Setup check (first run)

Before generating anything, check that each tool is available. For anything missing, say what it is and what it is for, then **ask before installing**. Never type passwords: the user stays signed in to Flow and Google in their own Chrome.

| Tool | Check | Install |
|---|---|---|
| Chrome with browser automation (for example Claude in Chrome) | the browser tools are listed and a tab opens | Install the extension and connect it |
| Flow account | `flow.google.com` opens signed in | The user signs in |
| `ffmpeg` | `ffmpeg -version` | Windows: `winget install Gyan.FFmpeg`. macOS: `brew install ffmpeg`. Linux: the package manager. |
| Node.js (for HyperFrames) | `node --version` | Install from nodejs.org; HyperFrames then runs with `npx hyperframes` |
| 4K upscaler | a Colab notebook with Real-ESRGAN is ready | Free cloud GPU notebook (Google Colab, L4 GPU): upload the 1K images, run Real-ESRGAN 4x, download the results. In the author's tests: about 4 s per image. Large zips can be split, or moved through Drive. |
| Download extension (optional) | installed in Chrome | Only for the backup download route |

Local upscaling on a weak laptop gave poor results, so prefer the cloud notebook. Opening the notebook and approving Google's permission prompts is done together with the user.

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
4. **Check for gaps.** Count the numbered files and report missing numbers. Flow's own 4K upscale takes about 25 s per image (300 files is roughly 2 to 2.5 hours), so do not run upscales in parallel.
5. **Storyboard.** Make 640 px previews with `ffmpeg`, label each with number, type and time, and compose them into sheets of 24. Missing items appear as gray boxes. Keep prompt order. Display the finished sheets to the user (open or attach the images): two sheets of 24 for 48 items. If you also publish a design page, set it to fill the page rather than using a canvas view, because the canvas view disappears in the side panel.
6. **Rough cut.** Scaffold a HyperFrames project (HyperFrames is an HTML-to-video tool, run with `npx hyperframes`) and:
   - take durations from the prompt labels;
   - use lightweight 1920 px preview copies (4K freezes low-memory machines);
   - cut audio from the narration track with the same time ranges and check it against the subtitle timings;
   - add a gentle Ken Burns move with 0.25 s fade in and out, and keep video clips muted;
   - run `npx hyperframes check --json`, then `npx hyperframes preview --background` and show it in the browser panel.
   By default no final render is taken here, and large files are not copied to cloud storage; switch to full-resolution files only if the user asks for a final render.

## Single image requests

Prerequisites: Chrome is open, the user is logged in to Flow, and the browser automation is connected. For "generate this one image": open a project, give the prompt to the agent, start it if it costs 0 credits, download the image when done (project menu or the optional extension), upscale it to 4K outside Flow if needed, and show the result. It is not instant: about 10-20 s to generate, plus upscaling time.

## Durations and credits (a snapshot)

Prices and models change, so this is a snapshot of what Flow showed when we checked (November 2026). The rule above still holds: read the price Flow displays, quote it, and ask before generating.

- **Omni 1.2 Flash:** 3 to 15 seconds, second by second. Flow's own model description lists it as the best quality model (picture quality, smoothness of motion, consistency over time, fidelity to a reference image).
- **Veo 3.1 Lite, Fast and Quality:** 4, 6 or 8 seconds only. Flow describes Veo 3.1 Quality as the best of the Veo family and the Omni models as better overall.
- **Video Extension** lengthens an existing video second by second.

Credits for Omni 1.2 Flash at 720p: 3 s = 9, 4 s = 12, 5 s = 15, 6 s = 18, 7 s = 21, 8 s = 24, 9 s = 27, 10 s = 30, 11 s = 36, 12 s = 42, 13 s = 48, 14 s = 54, 15 s = 60 (the price per second rises above 10 seconds). At 360p the price per second is lower (6 s = 6 credits, 15 s = 20). Veo 3.1 at 720p is a fixed price per generation: Lite 5, Fast 10, Quality 100.

Generate video at 720p (Flow's download gives the 720p original, not a scaled version) and bring it to 1080p afterwards with `ffmpeg` (so the 1080p clip is an upscale of the 720p original); always show the user the model, the duration and the total credits before generating, and never generate without approval.

## Lessons learned

- The Flow agent handles 24 items per run, and the bottom strip shows only that group.
- Local upscaling on a weak laptop gave poor or unusable results: plain `ffmpeg` Lanczos scaling took 1.7 s but looked low quality, and Real-ESRGAN on an integrated GPU took 103 s per image and left a black band. Use a free cloud GPU notebook instead (a paid cloud GPU would be about $0.005 per image).
- A separate automated browser (Playwright) never finished Flow's own upscale step; the real Chrome profile worked.
- Browser tabs freeze with lots of media. If screenshots time out, read the page text or use JavaScript, then reload.
- **Splatter and confetti come from the wording.** A style block with "rough sponge-spatter edges", or a prompt with "sparks, flung paint lines, scattered streaks", produces splatter. Write "clean painted edges, no splatter, no confetti, no scattered specks; dusty paper grain on every surface" into the style block and describe marks as drawn with a single brush stroke.
- **Colors come out darker than the hex you write.** An orange background requested as `#E4572E` measured about `#B35F3C` to `#BE653D`. Correcting the tone with an edit command ("lighter orange") did not work (measured, and credits were spent), so do not spend credits on tone corrections; approve the first tone you like.
- **Fixing a video.** The "Describe how to edit this video..." box on the clip page produces a corrected version of the same clip (it fixed confetti artifacts). It starts without a credit prompt (about 15 credits for 5 seconds in our run), so quote the cost to the user before you press it; the box shows no price, so give the table estimate and say it is an estimate. Download through the arrow, then "720p Original size" (free); the file is named `NAME_<code>.mp4`, so give it its prompt number again afterwards.
- **The chat panel.** Use "New session" at the top of the panel for a new chat session inside the same project (not a new project). Paste a long prompt through the clipboard (`Set-Clipboard` then Ctrl+V on Windows). Enter may not send; click the arrow at the bottom right. Several image prompts in a row work in the same session, an image name is given by writing "name it X" in the prompt, images were free in our run (Nano Banana 2.1), and 3 images took about a minute.
- **Side by side comparison.** The Images tab lists new images first, from left to right, newest to oldest. Capture a region with zoom and save it to disk, label the pieces with PIL, and show the result to the user.
- **Batch generation.** 12 scenes in one message worked without problems (24 items is the maximum per run, 12 was the size we proved) and took about 90 to 120 seconds. Every message needs its REFERENCE: a single collage reference (for example 4 liked examples in one JPG in Uploads, right-click, "Add to prompt"), or a character reference (Characters, the character, right-click, "Add to prompt"). The reference drops out of the prompt after every send, so add it again in every message. A generation without its reference counts as invalid.
- **Matching images to prompt numbers.** With several prompts in one message Flow does not rename the images to your numbers; it titles them from the content (the "name it X" trick above works for a single image), so the numbering comes from this matching. The `<img alt>` of the chat gallery carries the full prompt, so take a unique 50 character fingerprint from each scene's prompt and match it by scrolling the chat (the gallery shows at most about 6 thumbnails per message). Regenerate the missing ones in messages of 6 or fewer.
- **Browser traps.** A project page with hundreds of media can freeze the renderer: `screenshot` times out but `find` and JavaScript still work, so continue with reference-based clicks. JavaScript calls last about 45 seconds at most, so start long loops without awaiting them, and a JavaScript item is not accepted inside a `browser_batch`. A `fetch` to a localhost address hangs in Chrome (a local network permission); to get a file, use `fetch(src)`, then a blob and an `a[download]` link to save it into Downloads (webp at 1376x768 in our run). Strings that contain URLs or cookies are blocked as tool results, so do the matching inside the page.

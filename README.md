# Google Flow Video Generator

A Claude skill that runs a full AI image and video production line in Google Flow through browser automation: generate from a prompt file, number every output, download, find gaps, build a storyboard and a rough-cut preview. Every action that costs credits waits for your approval.

> Flow has no official API. This skill drives the Flow web app in your own logged-in Chrome session. Check Google's terms of use and accept the account risk before automating it.

## What problem it solves

Producing hundreds of scenes by hand in Flow is slow and easy to get wrong: lost numbering, forgotten character descriptions, accidental credit spending. This skill gives Claude a strict, repeatable routine.

## How it works

1. **Generate** from a prompt file in groups of 24. A shared "world vocabulary" block (the characters, symbols and visual style used by every scene) is pasted before every group, because Flow does not remember it between submissions.
2. **Approve credits yourself.** Claude shows the cost Flow displays and presses Approve only after your yes. It never chooses "always approve in project".
3. **Rename** every item with its prompt number (001, 002, ...), 24 at a time.
4. **Download** through the project zip (main route) or a helper extension (backup), upscale images to 4K outside Flow and bring videos to 1080p with `ffmpeg`. Video is never upscaled to 4K.
5. **Check for gaps:** numbered files are counted and missing numbers reported.
6. **Storyboard:** labeled 640 px previews in sheets of 24, missing items shown as gray boxes.
7. **Rough cut:** a HyperFrames project with durations taken from the prompt labels, light 1920 px previews, narration cut to the same time ranges, gentle Ken Burns motion and short fades.

## Guardrails

- One Flow project per video (old or heavy projects freeze the browser tab).
- No deleting or moving files without asking; browser `(1)` duplicates are listed first.
- No parallel downloads (Flow shows reCAPTCHA and upscale warnings).

## Requirements

- Claude Code with browser automation (for example Claude in Chrome)
- A Google Flow account, signed in through Chrome
- `ffmpeg`
- Optional: HyperFrames for the rough cut, a free cloud GPU notebook for Real-ESRGAN upscaling

## Install

Copy the `google-flow-video-generator` folder into `~/.claude/skills/`.

## Example prompts

- "Generate this prompt file in Flow."
- "Make the storyboard for the scenes we just generated."
- "Generate this one image in Flow."

## Files

- `SKILL.md`: the full workflow and rules Claude follows.

## License

MIT. See [LICENSE](LICENSE).

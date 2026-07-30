---
name: flow-broll-palmier
description: Use when the user asks to create images, videos, b-roll, scenes, or animations in Google Flow at labs.google/fx, including requests to choose a model, aspect ratio, quantity, or generation settings.
---

# Google Flow Media Agent

Use Google Flow through the available browser capability. Do not claim that a
generation succeeded unless the browser reports success and the resulting
media can be located or downloaded.

## Chrome control via OpenCode (built-in)

OpenCode can control Google Chrome through Chrome DevTools Protocol (CDP)
regardless of whether Cloud is enabled. This works with any OpenCode session
that has the chrome-devtools tools available:

| Tool | Purpose |
|---|---|
| `chrome-devtools_navigate_page` | Open a URL, reload, go back/forward |
| `chrome-devtools_click` | Click any element by its uid |
| `chrome-devtools_type_text` | Type text with real keystrokes into the focused element |
| `chrome-devtools_fill` / `fill_form` | Set input values / batch-fill forms |
| `chrome-devtools_take_snapshot` | Read the page's a11y tree (get uids and state) |
| `chrome-devtools_take_screenshot` | Capture a visual screenshot of the page |
| `chrome-devtools_list_network_requests` | Inspect network traffic |
| `chrome-devtools_upload_file` | Upload files from the local filesystem |
| `chrome-devtools_press_key` | Send keyboard shortcuts (Escape, Enter, etc.) |
| `chrome-devtools_wait_for` | Wait for text to appear on the page |
| `chrome-devtools_drag` | Drag and drop elements |
| `chrome-devtools_select_page` / `list_pages` | Manage multiple open tabs |
| `chrome-devtools_new_page` / `close_page` | Open and close tabs |

If the browser tool is not available in the session, follow the existing
handling in the [Failure handling](#failure-handling) section below.

## Before opening Flow

Ask only for details that are missing:

- What should appear in the image or video?
- Image or video?
- Visual style, camera movement, lighting, and language of any spoken audio?
- Aspect ratio: 16:9, 4:3, 1:1, 3:4, or 9:16.
- Number of results and approximate duration for video (4s, 6s, 8s, or 10s).
- Reference images, if any, and their local paths.

Preserve the user's wording and intent when turning the request into a prompt.
Do not invent people, brands, dialogue, or visual details unless asked.

## Default agent settings

Match the settings shown in the user's agent configuration when the user does
not specify alternatives:

- Ask for confirmation immediately before a generation that spends credits.
- Images: 16:9, 2 results, model `Nano Banana 2`.
- Videos: 16:9, 1 result, model `Omni Flash`.

The user may override any of these settings. Ask for confirmation before
changing a setting that materially changes credit usage or output type.

## Flow UI map (Canvas mode — preferred)

Prefer **Canvas mode** (Agente button NOT pressed). It is more reliable for
browser automation than Agent chat mode.

Bottom bar controls (left to right):

| Control | Label pattern | Role |
|---|---|---|
| Add media | `add_2 Criar` | Opens library dialog (existing assets only) |
| Mode toggle | `Agente` | Pressed = Agent chat; unpressed = Canvas |
| Settings chip | `Vídeo · 10s crop_16_9 x1` or `🍌 Nano Banana 2 crop_16_9 x2` | Opens generation settings menu |
| Submit | `arrow_forward Criar` | Starts generation (enabled only after valid prompt) |
| Prompt box | `O que você quer criar?` | Multiline textbox for the prompt |

### Settings chip menu (click the chip)

Tabs at top: `image Imagem` | `videocam Vídeo`

**Video settings when Vídeo is selected:**

- Mode tabs: `crop_free Frames` | `chrome_extension Elementos` (default Elementos for text-to-video)
- Aspect: `crop_16_9 16:9` | `crop_9_16 9:16`
- Model button: e.g. `Omni Flash arrow_drop_down`
- Duration tabs: `4s` | `6s` | `8s` | `10s`
- Result count: `x1` | `x2` | `x3` | `x4`
- Credit cost link shown at bottom (e.g. `15 créditos`)

**Image settings when Imagem is selected:**

- Aspect: `16:9` | `4:3` | `1:1` | `3:4` | `9:16`
- Model button: e.g. `🍌 Nano Banana 2`
- Result count: `x1`–`x4`

Close the menu with Escape before typing the prompt.

### Agent mode (fallback only)

Agent mode has its own panel with `tune Configurações` for defaults and a
chat-style prompt box. Prefer Canvas. Use Agent only if the user explicitly
asks for the agent, or Canvas submit fails after a correct prompt entry.

Agent settings panel (`tune Configurações`):

- Confirm before generate: Sempre / Nunca
- Image defaults and video defaults (same options as the chip menu)
- Save with `Salvar`

## Browser workflow

1. Open `https://labs.google/fx/pt/tools/flow` or the project URL supplied by
   the user.
2. Use the existing browser session if the browser tool exposes one. Never ask
   for or handle the user's Google password, cookies, or authentication tokens.
3. If Google requests login or a consent step, stop and ask the user to handle
   it in the visible browser. Continue only after the user confirms.
4. Click `Novo projeto` (or open an existing project). Wait until the project
   URL contains `/project/<id>`.
5. Ensure Canvas mode: if `Agente` is pressed, click it once so it is unpressed.
6. Click the settings chip (`Vídeo · …` or `🍌 …`). Select Imagem or Vídeo,
   then model, aspect ratio, duration (video), and result count. Escape to close.
7. Show the final settings and credit cost. Obtain confirmation if credits will
   be spent and confirmation is enabled.
8. Enter the prompt using the **prompt entry recipe** below.
9. Click `arrow_forward Criar` once. Do not repeatedly click Generate.
10. Wait for the canvas card progress to finish (slider disappears; card becomes
    playable). Open the card/edit link if needed.
11. Download only when the user asks for a local file. Click `download Baixar`
    on the editor page, then report the path under `~/Downloads/`. Otherwise
    report the project/result URL.

## Prompt entry recipe (critical)

React-controlled inputs ignore programmatic value sets (`fill` alone, native
value setters, synthetic input events without focus). Wrong entry causes
`Você precisa fornecer um comando` even when text is visible.

Correct sequence every time:

1. Click the prompt textbox so it is focused (`focusable focused`).
2. Type with real keystrokes (`type_text`), not only `fill`.
3. Snapshot and confirm:
   - Textbox value shows the full prompt (not the placeholder).
   - `arrow_forward Criar` is **not** `disableable disabled`.
   - Optional: `close Apagar comando` appears.
4. Only then click `arrow_forward Criar`.

If the button stays disabled or the error appears, clear the box, refocus,
retype with `type_text`, and retry once. Do not spam submit.

## Waiting for generation

- Canvas card shows progress (percent and/or slider). Video often takes 1–3+
  minutes; keep waiting without re-clicking Generate.
- Progress may jump around; that is normal.
- Done when the progress slider is gone and the card has play controls, or the
  editor shows duration (e.g. `00:10:00`) and `download Baixar` is enabled.
- Failure shows `warning Falha` on the card. Report it and offer to reuse the
  command (`undo` / `redo Reutilizar comando`) or adjust the prompt.

## Prompt structure

For video, build prompts in this order:

`subject and action; setting; composition and camera; lighting and palette;
style and realism; motion and timing; audio/dialogue if requested; exclusions.`

For images, omit timing and motion unless they describe the desired visual
effect. Keep prompts concise enough for the Flow interface while retaining
the user's required details.

### Narrative / time transitions

When the user wants a short arc (e.g. “from catching the plane to lounging in
Paris”), write one continuous beat, not hard cuts:

- Good: `a young woman boards a plane, then later lounges lying down in Paris…`
- Avoid: multi-scene screenplay formatting that fights a single 4–10s clip.

Preserve intent; compress into one flowing action chain that fits the duration.

### Prompt quality checklist

- Keep the user’s subject, place, era/style, and must-have actions.
- Prefer concrete camera language: wide shot, handheld, slow push-in.
- Put audio last: `2000s pop music bed`, `no dialogue`, etc.
- Drop filler adjectives that do not change the frame.
- Match language to the UI when helpful (PT-BR UI accepts PT prompts well).

## Failure handling

- If the browser tool is unavailable, say that this OpenCode session has no
  browser capability configured. Do not pretend that a shell command opened or
  controlled Flow.
- If selectors or labels differ, inspect the current page and use visible
  labels rather than brittle coordinates or fixed uids.
- If a quota, credit, region, or login error appears, report it verbatim and
  ask the user how to proceed.
- On `Você precisa fornecer um comando`, fix prompt entry (recipe above); do
  not assume the model or credits failed.
- Never bypass CAPTCHA, account security, access controls, or payment prompts.
- Do not delete user downloads unless the user asks.

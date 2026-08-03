# Download X (Twitter) Videos — iOS Shortcut

A step-by-step guide to building an iOS Shortcut that saves the video from any
public X.com post to your iPhone.

> **Why instructions instead of a ready-made `.shortcut` file?**
> Apple's Shortcuts app only runs on iOS/iPadOS, and shortcut files are signed
> per-device when exported from the app. A hand-built `.shortcut` file authored
> outside the app can silently break (wrong action IDs, malformed variable
> references) with no way to test it here. Building it directly in the app
> takes about 3–5 minutes and is guaranteed to work. Two methods are below —
> pick one.

> **Use responsibly.** Only save videos you have the right to keep (your own
> posts, or ones the creator is fine with you downloading) and don't
> redistribute copyrighted content. This uses public, unauthenticated
> endpoints — no login or API key required — but X's Terms of Service place
> restrictions on automated access, so this is offered for personal,
> occasional use at your own discretion.

---

## Method A — Quick & reliable (recommended)

Two actions. Hands off the actual video extraction to a web page, so nothing
here can go stale from an API changing shape.

1. Open the **Shortcuts** app → tap **+** to create a new shortcut.
2. Add action **Ask for Input**:
   - Input Type: `URL`
   - Prompt: `X post link`
   - Tap the blue **"Default Answer"** field → **Select Variable** → choose
     **Shortcut Input**. (This makes the prompt auto-fill when you invoke the
     shortcut from the Share Sheet, so you can just tap Done.)
3. Add action **Text**. In the text field type:
   ```
   https://twittervideodownloader.com/?url=
   ```
   Then tap inside the field where your cursor is at the end, tap the
   **variable picker** (the small icon above the keyboard / the "..." menu),
   and insert the **Provided Input** variable (the output of the Ask for
   Input step) right after the `=`.
4. Add action **Open URLs**, and set its input to the **Text** result from
   step 3 (tap the field → pick the Text variable).
5. Tap the shortcut's name at the top → rename it, e.g. **"Download X
   Video"**.
6. Tap the settings icon (ⓘ) → enable **Show in Share Sheet**, and under
   **Share Sheet Types** restrict it to **URLs** (and optionally **Safari web
   pages**).
7. Save.

**Using it:** In the X app (or Safari), open the post with the video, tap
**Share** → **Download X Video**. It opens a page with direct download
links for the video — tap the quality you want, then use the Safari
download button (or "Save to Files") to save it. From Files you can move it
into Photos.

---

## Method B — One-tap save straight to Photos (more advanced)

More actions, but the video lands directly in your Photos library without a
browser detour. Uses the free, public `fxtwitter.com` API (a well-known
open-source project that mirrors X post data, including direct video URLs,
for embedding purposes). Because it's a third-party mirror rather than an
official API, treat it as "usually works" rather than guaranteed forever —
fall back to Method A if it ever stops responding.

1. Open **Shortcuts** → **+** for a new shortcut.
2. **Ask for Input** — Input Type `URL`, Prompt `X post link`, Default Answer
   = **Shortcut Input** (same as Method A, step 2).
3. Add action **Match Text**:
   - Text: the output of Ask for Input (**Provided Input**)
   - Regular Expression: `\d{6,}`
   This pulls the numeric tweet ID out of the URL (works for both
   `x.com/.../status/123...` and `twitter.com/.../status/123...` links).
4. Add action **Get Item from List**, set to **First Item**, with input the
   **Matches** from step 3. This is your Tweet ID.
5. Add action **Text**, content:
   ```
   https://api.fxtwitter.com/status/
   ```
   then insert the Tweet ID variable right after it (same trick as Method A
   step 3) to build the API URL.
6. Add action **Get Contents of URL**:
   - URL: the Text result from step 5
   - Method: GET (default)
7. Add action **Get Dictionary Value**, key `tweet`, input = the previous
   step's result (Shortcuts treats JSON responses as a dictionary
   automatically).
8. Add another **Get Dictionary Value**, key `media`, input = previous step.
9. Add another **Get Dictionary Value**, key `videos`, input = previous step.
   This gives a list of available video variants.
10. Add **Get Item from List** → **Last Item** (fxtwitter lists variants
    lowest→highest bitrate, so Last Item is the best quality) on the list
    from step 9.
11. Add **Get Dictionary Value**, key `url`, input = previous step. This is
    the direct `.mp4` URL.
12. Add **Get Contents of URL** again, URL = previous step's result, to
    actually download the video file.
13. Add action **Save to Photo Album** (or **Save File** if you'd rather
    keep it in Files), input = the file from step 12.
14. Name it (e.g. **"Save X Video"**), enable **Show in Share Sheet** →
    restrict to **URLs** as in Method A.

**Using it:** Share a post with a video → **Save X Video** → confirm the
Photos save prompt. Done — no browser step.

**If a post has no video** (photo-only or link post), step 9's list will be
empty and the shortcut will fail at step 10/11 — that's expected; there's
nothing to download.

---

## Troubleshooting

- **"Untrusted Shortcut" warning on first run:** normal for shortcuts you
  build yourself the first time you run one containing "Get Contents of
  URL" against the internet — tap **Trust** (or enable it once under
  Settings → Shortcuts → Advanced → Allow Untrusted Shortcuts if prompted at
  the OS level).
- **Method B stops working:** the fxtwitter mirror occasionally changes its
  response shape or goes down; use Method A in the meantime.
- **Video quality:** Method B's step 10 grabs the highest-bitrate variant
  fxtwitter reports; Method A lets you pick the quality manually on the
  download page.
- **GIFs / multi-video posts:** these guides target the common single-video
  post case; posts with multiple videos will only return the first/last
  match depending on the method.

# Faculty number tool

A tiny web page that hands out a **unique number that is never repeated**, starting at
**4000**. Built for two teachers who each need to pull numbers from one shared sequence,
on their own devices, with no login.

- **The page** (`index.html`) is a single static file. It is served by GitHub Pages.
- **The counter** is one small online service both teachers call. It holds "the next
  number" and increments it atomically, so the same number can never go out twice.

The page and the counter are separate because GitHub Pages can only serve static files,
it cannot remember a number between visitors. The counter's source still lives here in
this repo, so GitHub stays the home of record for all the code.

```
index.html                     the page (this is what GitHub Pages serves)
counter/apps-script/Code.gs    the counter  (main option: Google Apps Script)
counter/cloudflare/worker.js   the counter  (backup option: Cloudflare Worker)
```

---

## How it fits together

```
Teacher A's browser  \
                      >--->  one shared counter  --->  4000, 4001, 4002, ...
Teacher B's browser  /        (Apps Script)
```

Both copies of the page (yours and the CTLS one) point at the **same** counter URL, which
is what makes it one shared sequence across both teachers and both sites.

---

## Setup, in order

### 1. Put the code on your GitHub

1. Create a new repository on your GitHub account (for example `faculty-number-tool`).
2. Upload the contents of this `repo/` folder to it.
3. Repo **Settings -> Pages -> Build and deployment -> Source: Deploy from a branch**,
   branch `main`, folder `/ (root)`. Save.
4. After a minute your page is live at `https://<your-username>.github.io/faculty-number-tool/`.
   It will show a yellow **"Demo mode"** banner until step 2 is done. That is expected and safe.

### 2. Deploy the counter (Google Apps Script)

1. Go to [script.google.com](https://script.google.com) -> **New project**.
2. Delete the sample code and paste in `counter/apps-script/Code.gs`.
3. **Deploy -> New deployment -> Web app**:
   - Execute as: **Me**
   - Who has access: **Anyone**
4. Copy the **web app URL** it gives you (ends in `/exec`).
5. In `index.html`, paste that URL between the quotes here:
   ```js
   const ENDPOINT = "https://script.google.com/macros/s/AKfy.../exec";
   ```
6. Commit that change. The "Demo mode" banner disappears and the tool is live and shared.

> Tip: visit the `/exec` URL in a browser to see the current status. It is safe, a plain
> visit does **not** use up a number (only the page's button does).

### 3. Stand up the page on the CTLS org GitHub

Once it works on your account, repeat step 1 under the **CTLS organisation** (same files,
same `ENDPOINT` URL so both sites share the one counter). Done.

---

## Changing things

- **Different starting number:** change `START` in `index.html` *and* `START_AT` in the
  counter, before anyone uses it. To reset later, run `resetCounterTo()` in the Apps
  Script editor.
- **Wording / colours:** all in `index.html`. The palette is the RDP softened brand
  (forest `#004238`, lime `#AADB1E`).
- **If Apps Script ever gives trouble** (rare cross-origin issue): switch to the Cloudflare
  Worker in `counter/cloudflare/` and point `ENDPOINT` at it instead. Nothing else changes.

## Why it never repeats

The counter increments under a lock, so requests are handled one at a time. If a teacher's
internet drops mid-request, the page refuses to show a number rather than guessing, so a
network blip can never cause a duplicate. The "Recent on this device" line on the page is
just a convenience and is not the source of truth, the counter is.

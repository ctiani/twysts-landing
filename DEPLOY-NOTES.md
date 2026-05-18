# Deploy notes — TWYSTS Protein+ landing page

> STATUS (2026-05-18): **DECIDED — GitHub Pages + Web3Forms.**
> Production domain is **`www.twysts.com`**, set directly as the GitHub
> Pages custom domain (no `twysts.pointpixl.com` middle hop — chaining a
> CNAME through it breaks GitHub's SSL). Staging URL is the repo's
> `https://<user>.github.io/<repo>/`. Client IT adds ONE DNS record when
> ready (below). Form submits client-side to Web3Forms — no backend.
> The earlier Cloudflare Pages + Resend function was removed on 2026-05-18.

## Stack
- **Host:** GitHub Pages (static, free; repo must be public OR a paid GitHub plan that allows Pages on private repos)
- **Staging URL:** `https://<user>.github.io/<repo>/` (Pointpixl-controlled, always available)
- **Production URL:** `www.twysts.com` (GitHub Pages custom domain; client adds one CNAME)
- **Form handler:** Web3Forms (AJAX) — wired in `index.html`, no backend
- `.nojekyll` at repo root disables GitHub's Jekyll build (plain static site)

## Form — what's wired
- The lead form (`.form-card` in `index.html`) keeps the full custom
  frontend: client-side validation, `website` honeypot, success state,
  pretzel waterfall, height-lock, smooth scroll. None of that changed.
- On submit it POSTs JSON to `https://api.web3forms.com/submit` with:
  - `access_key` (public by design — tied to the destination inbox, not a secret)
  - `subject`: `TWYSTS lead: NAME from COMPANY`
  - `from_name`: `TWYSTS Protein+ Landing Page`
  - `replyto`: the lead's email (so Reply in the lead email goes to them)
  - the four fields: name, company, email, role
- Success state + waterfall fire regardless of POST result (never block the
  user). **Always verify via the Network tab + the actual inbox, not the
  on-screen confirmation.**
- Spam: client honeypot + Web3Forms server-side filtering. Add hCaptcha
  later if the show generates abuse.

### Web3Forms key in use
- **TEST key:** `7c04f440-4fe3-4531-b394-2a2349801c7e` — currently in
  `index.html`, allowed domain set to `localhost`, leads go to the test inbox.

### Web3Forms domain + production checklist
1. **For staging tests** on `https://<user>.github.io/...`: add
   `<user>.github.io` to the form's allowed domains (the Origin header is the
   bare github.io host, not the repo path). Keep `localhost` too.
2. **Before launch**: add `www.twysts.com` (and `twysts.com` if apex is used)
   to allowed domains.
3. Set the destination email to **`info@thepretzelgroup.com`** (currently a
   test inbox). Either reconfigure the existing form or create a production
   form and swap its access key into `index.html` (current key:
   `7c04f440-4fe3-4531-b394-2a2349801c7e`).
4. Re-test from the live URL: submit, confirm `200` + `{"success":true}` in
   the Network tab, confirm the email arrives, confirm Reply goes to the lead.

## GitHub Pages setup

1. Push the repo (site files only — see `.gitignore`; verified 41 files, 5.6MB).
2. Repo → **Settings → Pages** → Source: **Deploy from a branch** →
   Branch: `main`, folder: `/ (root)` → Save.
3. Site goes live at **`https://<user>.github.io/<repo>/`** in ~1 min.
   This is the staging URL — test everything here first.
4. **Do NOT add the custom domain or commit a `CNAME` file until the client
   is ready to cut DNS.** Setting it early makes Pages 301-redirect the
   github.io URL to `www.twysts.com`, which won't resolve yet → dead preview.
5. When ready for production: Settings → Pages → Custom domain →
   enter `www.twysts.com` → Save. GitHub writes a `CNAME` file and begins
   SSL provisioning (Let's Encrypt; minutes, up to 24h). Tick
   "Enforce HTTPS" once the cert is issued.

## DNS record to hand the client (for www.twysts.com)

ONE record. Give the client's IT exactly this:

| Host record      | Type  | Value               |
| ---------------- | ----- | ------------------- |
| `www.twysts.com` | CNAME | `<user>.github.io`  |

(Replace `<user>` with the GitHub account/org that owns the repo — e.g.
`pointpixl.github.io`. Note: it's the bare `<user>.github.io`, NOT the repo
path.)

### Apex `twysts.com` (optional but recommended)
If they also want bare `twysts.com` to work, the client adds A records to
GitHub Pages' IPs and GitHub redirects apex → the www custom domain:

| Host record    | Type | Value           |
| -------------- | ---- | --------------- |
| `twysts.com`   | A    | `185.199.108.153` |
| `twysts.com`   | A    | `185.199.109.153` |
| `twysts.com`   | A    | `185.199.110.153` |
| `twysts.com`   | A    | `185.199.111.153` |

(GitHub Pages' published apex IPs as of 2026 — confirm current values at
docs.github.com/pages before sending if there's any doubt.) Alternatively,
the client's registrar can just 301-redirect `twysts.com` → `www.twysts.com`.

Notes for the client:
- Propagation: minutes to a few hours. GitHub SSL issues automatically
  once the CNAME resolves to `<user>.github.io`.
- Until they make this change, the site stays reachable only at the
  staging github.io URL — zero impact on their current DNS.

## Remaining punch list (not deploy-blocking, but pre-launch)
- **Sales-sheet PDF:** DONE — wired to `Art/in-use/twysts-sellsheet.pdf`
  (success-state download button) with `Art/in-use/twysts-sellsheet.webp`
  as the on-page preview. Swap the file if the client sends a revised sheet.
- **Analytics:** `<head>` of `index.html` has an empty placeholder block for
  GA4 + Meta Pixel. Insert tags when measurement IDs are provided.
- **Web3Forms production** — see checklist above. Do not launch on the test
  key / test inbox.

## What to commit to the repo (site files only)
The live page loads everything from `Art/in-use/` (WebP, ~5.6MB total).
The 173MB `Art/Packaging/`, `Art/Logos/`, `Source/`, etc. are NOT used.

Include: `index.html`, `privacy.html`, `terms.html`, `favicon.svg`,
`Art/in-use/` (Fonts, Packaging WebP, SVG, sell-sheet PDF+WebP),
`DEPLOY-NOTES.md`, `.gitignore`.
Exclude (handled by `.gitignore`): `Source/`, `Documents/`, `Design/`,
`Proofs/`, `Production/`, `Research/`, `Copy/`, `Art/Packaging/`,
`Art/Logos/`, `Art/Raw/`, `Art/Stock/`, `*.zip`, `wireframe.html`,
`claude-code-prompt.md`, `.DS_Store`, `dist/`, `.claude/`.

## How to verify after deploy
1. Open `https://twysts.pointpixl.com/` (then the client domain once DNS is in).
2. DevTools → Network. Submit the form with test data.
3. Expect `POST https://api.web3forms.com/submit` → `200` → `{"success":true}`.
4. Confirm the lead email arrives at the configured inbox within ~30s,
   subject `TWYSTS lead: NAME from COMPANY`, Reply addressed to the lead.
5. Console may show `A listener indicated an asynchronous response…` — that
   is a browser-extension artifact, not site code. Ignore (gone in Incognito).

## Troubleshooting
- **Web3Forms returns `{"success":false}` / 403-ish** → submitting origin
  isn't in the form's allowed-domains list. Add the live hostname.
- **Form shows success but no email** → check Network tab; if the POST
  failed, the success UI still showed (by design). Fix the allowed domain or
  access key.
- **Apex domain won't resolve** → provider doesn't support ALIAS/ANAME or
  CNAME flattening; switch to A records or move apex DNS to a provider that does.
- **SSL warning on the client domain** → the host doesn't have `twysts.com` /
  `www.twysts.com` added as custom domains yet; add them so it can issue certs.

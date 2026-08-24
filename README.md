# Leaderboard + micro back office

Five files, no build step, no dependencies. Runs on GitHub Pages.

| File | Purpose |
|---|---|
| `index.html` | Public leaderboard. Loads `data.json` once per page load. |
| `admin.html` | Back office: password sign-in, edit names and positions, publish. |
| `data.json` | The only data store. |
| `logo.png` | Turbo Stars logo, white on transparent. |
| `DEPLOY.md` | Step-by-step Git and GitHub Pages guide. |

---

## 1. Deploy

1. Create a repository (**Private** — see "Known weaknesses").
2. Put all five files in the root.
3. Settings → Pages → Source: `Deploy from a branch` → branch `main`, folder `/ (root)`.
4. After a minute or two:
   - leaderboard — `https://<owner>.github.io/<repo>/`
   - back office — `https://<owner>.github.io/<repo>/admin.html`

> Pages on a private repository needs GitHub Pro or Team. On the free plan Pages only serves from a public repo, which means `admin.html` is visible to anyone and the password is the only barrier.

## 2. Sign-in credentials

Login `edu-tournament`, password `edu-tournament-12987!jfk`.

They are stored as a single SHA-256 hash of `login\npassword` in `AUTH_HASH` near the top of the script in `admin.html`. To change them: sign in → "Change sign-in credentials" → enter the new pair → Generate hash → paste the value into `AUTH_HASH` → commit.

## 3. Connect one-click publishing

Without this the back office can only download `data.json` for you to upload by hand.

1. GitHub → Settings → Developer settings → **Fine-grained personal access token**.
2. Repository access: **this repository only**. Permissions: **Contents → Read and write**. Shortest workable expiry.
3. In the back office open "GitHub connection", fill in owner / repo / branch / path and paste the token.

The token lives in `sessionStorage` — current tab only, gone when the tab closes, never written to the repository. Repo settings (without the token) persist in the browser.

## 4. Day-to-day

Add a participant → type the name, nickname or email → set the position → **Publish update**. Reorder by dragging the `⠿` handle, then press **Renumber 1…N**.

**There is no auto-refresh.** The public page reads `data.json` once when it loads. Publishing from the back office replaces that file; anyone who opens the page after that gets the new standings. Someone already on the page sees the update after reloading it in their browser.

The `▲ 2` / `▼ 1` arrows are computed automatically as the difference against each participant's position at the previous publish. New entries show `NEW`.

## 5. Brand colours

Both files start with a `:root` block. The accent lives in one variable:

```css
--brand:#FF6A00;
```

Change it in `index.html` and `admin.html` and every accented element follows — the leader plate, focus rings, primary buttons, search highlight. Everything else is black, white and grey by design: the orange means "leader" and nothing else.

---

## Known weaknesses — read before launch

**The credentials in `admin.html` are a gate latch, not a lock.** GitHub Pages is static hosting; there is no server to check anything. The hash sits in the page source, so the only thing protecting it is password length. The current password is long enough that offline brute force is impractical — keep it that way if you ever change it.

What follows from that:

- **Can anyone overwrite `data.json`?** No. Writing to the repository is guarded by the GitHub token, not by the page password. Cracking the password gets someone the editor UI, not the ability to publish, unless the token is saved in the same browser.
- **Public emails are personal data.** Publishing a list of participant email addresses openly, without consent, is a direct GDPR exposure — more so for an iGaming brand. Publish nicknames. Real names carry the same issue, with consent.
- **The token in the browser.** If the back office is opened on someone else's machine and the tab stays open, the token stays with it. "Forget token" clears it immediately.

**Good enough for:** an internal or semi-public tournament, tens of participants, updates once a day, no money tied to the standings.

**Time to move off it when:** the table drives payouts, participants run into the hundreds, updates are frequent, or you need an audit trail of who changed what. Then use Cloudflare Pages + Access (SSO instead of a password) with the data in KV, or Supabase — real authentication, roles, change history. `index.html` carries over almost unchanged; only the data source moves.

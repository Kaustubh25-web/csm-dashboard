# CSM Dashboard (Essen Vision)

A Salesforce-style CSM dashboard that runs entirely on GitHub:

- **GitHub Pages** serves the app (`index.html`, a single static page with no build step).
- **A private GitHub repository** stores the shared data as one JSON file (`data/csm-data.json`). Every save is a commit, so you get a full change history and can roll back anything.
- Each CSM signs in with their **own fine-grained personal access token**. Edits are attributed to their GitHub username.

No server, database or other service is involved.

---

## 1. Create two repositories

| Repo | Visibility | Contains |
|---|---|---|
| `csm-dashboard` | Public (or private on a paid plan) | This folder: `index.html`, `.nojekyll`, `README.md` |
| `csm-data` | **Private** | Nothing to start with. The app creates `data/csm-data.json` on the first save. Add a README so the `main` branch exists. |

> Keep the data repo **private**. A GitHub Pages site is public on Free/Pro/Team plans, so the page itself holds no customer data. All data is fetched from the private repo using each user's token.

## 2. Publish the app with GitHub Pages

1. Push the contents of this folder to `csm-dashboard` (or drag and drop them via **Add file → Upload files**).
2. Open **Settings → Pages → Build and deployment**, then set **Source: Deploy from a branch**, **Branch: `main` / root**, and click **Save**.
3. After about a minute the site is live at `https://<owner>.github.io/csm-dashboard/`.

**Optional:** open `index.html`, find the `CONFIG` line near the top of the script, and pre-fill the data repo so CSMs only need to paste a token:

```js
const CONFIG = { owner: "essenvision", repo: "csm-data", branch: "main", path: "data/csm-data.json" };
```

## 3. Give each CSM access

1. Add each CSM to the `csm-data` repo (or org team) with **Write** access. Give **Read** access to anyone who should only view.
2. Each CSM creates a **fine-grained personal access token**: GitHub → **Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token**.
   - **Resource owner:** the user or org that owns `csm-data`
   - **Repository access:** *Only select repositories* → `csm-data`
   - **Permissions → Repository → Contents:** **Read and write** (Metadata: read is added automatically)
   - **Expiration:** your policy (for example 90 days)
   - If the repo belongs to an organization, an org owner may need to approve the token (**Org settings → Personal access tokens → Pending requests**).
3. The CSM opens the site, goes to **Data → GitHub connection**, pastes the token and clicks **Connect**.

The token is stored only in that browser (localStorage) and is sent only to `api.github.com`. Click **Disconnect** to remove it.

## 4. Load your data

On **Data → Upload master sheet**, drop the CSM Excel workbook. The app reads these tabs: *Master Account Tracker, Meetings, QBR Tracker, Monthly Cadence Tracker, Tower map, Ideas to Pitch Assessments, Account Towers - POC Mapping*. The whole upload is saved as **one commit**.

- Accounts are matched by name. Filled cells overwrite saved values; blank cells never erase them.
- Meetings are de-duplicated, so re-uploading the same file is safe.
- **Download Excel** exports everything back in the same tab layout.

If someone uses the page before connecting, their work is kept in the browser. It is copied into the repo automatically the first time they connect, but only if the repo has no data yet.

## How saving works

- Edits are batched (about one commit per second of activity) and written with the GitHub Contents API.
- If two CSMs save at the same moment, the app re-reads the file, re-applies only the fields each person changed, and retries. Both changes are kept.
- The page checks for other people's changes every 30 seconds, and whenever the tab regains focus. You can also click **Refresh now**.
- The header shows the save status: *All changes saved*, *Saving to GitHub…*, or *Sync error · retry*.
- To view or undo a change, use **Change history ↗**, which opens the commit list for the data file.

## Limits and notes

- The GitHub API allows 5,000 requests per hour per token, which is plenty for a team of CSMs.
- One data file is fine for thousands of meetings. GitHub's limit for a file edited this way is 100 MB.
- Excel import and export use SheetJS, loaded from `cdnjs.cloudflare.com`. Everything else is self-contained.
- To work on the app locally, run `python3 -m http.server` in this folder and open `http://localhost:8000`.

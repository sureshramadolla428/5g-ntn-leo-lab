# Push this pack to GitHub

Local `git init` and the first commit on **main** are already done. **Do not push until you intend to publish.**

Replace `<your-username>` with your GitHub account. Suggested repository name: `5g-ntn-leo-lab`.

## 1. Create an empty GitHub repo

On GitHub: **New repository** → name `5g-ntn-leo-lab` → **no** README / license / gitignore (this pack already has them).

Or with GitHub CLI:

```powershell
cd C:\Users\sures\OneDrive\Desktop\5g-ntn-leo-lab
gh repo create 5g-ntn-leo-lab --public --source=. --remote=origin --push
```

`--push` publishes immediately. If you want to review first, omit `--push` and use the commands below.

## 2. Add remote and push `main` (no force)

```powershell
cd C:\Users\sures\OneDrive\Desktop\5g-ntn-leo-lab
git remote add origin https://github.com/<your-username>/5g-ntn-leo-lab.git
git branch -M main
git push -u origin main
```

SSH remote (if you use keys):

```powershell
git remote add origin git@github.com:<your-username>/5g-ntn-leo-lab.git
git branch -M main
git push -u origin main
```

## 3. After the first push

- Confirm `HONESTY.md` is visible on the repo home (linked from README).
- Demo media in `docs/media/` is already in git (~42 MB `.webm` + Grafana Word doc). Do not add pcaps or VM logs.
- Dual-DU kit: still **not filed** upstream unless you separately paste `OAI_ISSUE_RFSIM_DUAL_DU_HO/ISSUE.md` on the OAI tracker.
- Keep `5g-ntn-leo-lab-scripts` **private**. Do not push that companion as a public repo.

## Do not

- `git push --force` to `main`
- Rewrite history after others have cloned
- Commit `.env`, keys, or `*.pcap`

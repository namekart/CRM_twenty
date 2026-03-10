# Remove README from GitHub (keep locally)

To stop showing the root README on GitHub but keep the file on your computer:

## 1. Stop tracking the root README and push

Run in PowerShell from the repo root (`E:\namekart\CRM\twenty`):

```powershell
git rm --cached README.md
git commit -m "Stop tracking root README (keep locally)"
git push
```

- **`git rm --cached README.md`** – Removes `README.md` from Git’s index only (file stays on disk).
- After you **commit** and **push**, `README.md` will be **removed from GitHub** and no longer show on the repo.
- The file remains in your local folder. Because `README` and `README.*` are in `.gitignore`, Git will not track it again.

## 2. (Optional) Remove all README files from GitHub

To remove every README from the repo on GitHub but keep all of them locally:

```powershell
git rm -r --cached --ignore-unmatch README.md
git ls-files | Select-String "README" | ForEach-Object { git rm --cached $_ }
git commit -m "Stop tracking all README files (keep locally)"
git push
```

Or on Linux/macOS/Git Bash:

```bash
git ls-files | grep -i readme | xargs git rm --cached
git commit -m "Stop tracking all README files (keep locally)"
git push
```

Use **section 1** if you only want the root README removed from GitHub.

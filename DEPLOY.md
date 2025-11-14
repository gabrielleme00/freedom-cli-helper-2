This repository is configured to deploy to GitHub Pages using GitHub Actions.

Follow these steps locally to create the remote repo and push your code:

1. Initialize git (if not already)

   git init
   git add .
   git commit -m "chore: initial commit"

2. Create the remote repository on GitHub (replace <owner> and <repo> with your values). You can run this with the GitHub CLI or create it in the web UI.

   # Using GitHub CLI (gh):
   gh repo create <owner>/<repo> --public --source=. --remote=origin --push

   # OR create via web UI and then add remote:
   git remote add origin https://github.com/<owner>/<repo>.git
   git branch -M main
   git push -u origin main

3. After pushing, GitHub Actions will build and deploy the site automatically. The workflow file is at `.github/workflows/deploy-pages.yml` and will deploy the `dist/` folder to GitHub Pages.

4. Verify the Pages site: go to the repository Settings -> Pages, or check the Actions tab for the "Build and Deploy to GitHub Pages" workflow run.

Notes and tips:
- The Vite config uses `base: './'` so assets are referenced relatively and work when the site is served from Pages.
- If your repository uses a different default branch name, adapt the workflow trigger and push commands accordingly.

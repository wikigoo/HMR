# Astro Backup Guide

### 1. Source Code Backup (GitHub)

* **Explanation:** The safest and most standard way to backup your project's source code is to use the Git system and the GitHub platform. These commands first prepare your project for Git, commit the changes, and finally push them to GitHub. Note that before running these commands, you must create a new, empty repository on GitHub and replace the URL of the repository with `<YOUR_GITHUB_REPO_URL>` on the fifth line.
* **Environment:** VSCode Terminal (PowerShell or CMD)
* **Path:** `D:/HMR-Astro`
* **Code:**

```bash
git init
git add .
git commit -m "Initial project backup"
git branch -M main
git remote add origin <YOUR_GITHUB_REPO_URL>
git push -u origin main

```

### 2. Cloudflare KV Backup (Session Data)

* **Explanation:** Your code was backed up in the previous step, but the session data is stored in the Cloudflare KV cloud. The `Wrangler` command line tool allows you to connect to your Cloudflare account and manage your storage spaces.
The following commands will first connect you to your Cloudflare account and then display a list of your KV spaces. Since you are a beginner and Wrangler does not have a direct command to download all the data in one click, the easiest way is to run these commands and ensure the connection, log in to the Cloudflare web dashboard, go to `Workers & Pages` > `KV` and manually view or export the session space data.
* **Environment:** VSCode Terminal (PowerShell or CMD)
* **Path:** `D:/HMR-Astro`
* **Code:**

```bash
npx wrangler login
npx wrangler kv:namespace list

````

### 3. Local Offline Archive (Zip Backup)

* **Explanation:** It is always useful to have an offline zipped copy on your PC for safety. The following command will create a zipped file of your entire project folder and save it to the root of your D drive.
*Very important note:* The `node_modules` folder is very large and does not need to be backed up. I strongly recommend that you manually delete the `node_modules` folder from your project before running this command (it will be recreated later whenever needed by running the `npm install` command).
* **Environment:** PowerShell
* **Path:** `D:/`
* **Code:**

```powershell
Compress-Archive -Path D:/HMR-Astro -DestinationPath D:/HMR-Astro-Local-Backup.zip

````

---

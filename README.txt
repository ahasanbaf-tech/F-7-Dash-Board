F-7 FLEET OPS BOARD — HOSTING & SETUP NOTES
=============================================

WHAT'S IN THIS ZIP
-------------------
index.html   -> the entire dashboard (HTML + CSS + JS in one file).
               Everything works offline / locally EXCEPT Google Drive sync,
               which requires this file to be hosted at a real http/https
               URL (not opened directly from disk).

HOW TO HOST IT
--------------
Any static host works. A few easy free options:

  GitHub Pages
    1. Create a new GitHub repository (public or private).
    2. Upload index.html to the repo (root, or a /docs folder).
    3. Repo Settings -> Pages -> Source: choose the branch/folder above.
    4. GitHub gives you a URL like:
       https://yourusername.github.io/your-repo-name/
    5. Share that URL with your maintenance crew.

  Netlify / Vercel / Cloudflare Pages
    Drag-and-drop the extracted folder (or just index.html) onto their
    dashboard's "deploy" screen — all three offer a free static-site tier
    with no build step needed for a single HTML file.

ENABLING GOOGLE DRIVE SYNC (so all devices/crew share one live file)
---------------------------------------------------------------------
1. Go to https://console.cloud.google.com and create a project (or use one
   you already have).
2. APIs & Services -> OAuth consent screen
     - User type: External
     - Publishing status: Testing is fine for a small crew
     - Under "Test users", add every crew member's Google account
3. APIs & Services -> Credentials -> Create Credentials -> OAuth client ID
     - Application type: Web application
     - Authorized JavaScript origins: add the exact URL you're hosting on,
       e.g. https://yourusername.github.io  (no trailing slash, no path)
     - Save, then copy the generated Client ID
       (it ends in .apps.googleusercontent.com)
4. Open index.html in a text editor, find this line near the top of the
   <script> block:

       var GOOGLE_CLIENT_ID = "PASTE_YOUR_OAUTH_CLIENT_ID_HERE.apps.googleusercontent.com";

   Replace the placeholder with the Client ID you copied.
5. Re-upload/re-deploy index.html to your host.
6. Open the hosted link, click "Connect Google Drive", sign in with one of
   the approved test-user accounts. Every crew member who signs in with an
   approved account will read/write the same data file
   (f7-fleet-ops-data.json) in Google Drive, kept in sync automatically.

NOTES
-----
- If you skip the Google Drive setup, the dashboard still works perfectly
  fine on a single device — it just falls back to that browser's local
  storage only (no cross-device sync).
- The PDF and Excel export buttons load small libraries from a public CDN
  the first time they're used — this needs an internet connection, but no
  further setup.
- All fleet, sortie, turnaround, and maintenance history is kept locally
  and (if connected) in your Drive file — nothing is sent to Anthropic or
  any third party other than Google, and only if you choose to connect.

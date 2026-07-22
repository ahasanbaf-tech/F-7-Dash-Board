F-7 FLEET OPS BOARD — HOSTING & SETUP NOTES
=============================================

WHAT'S IN THIS ZIP
-------------------
index.html   -> the entire dashboard (HTML + CSS + JS in one file).
               Everything works fully offline / on a single device with no
               setup at all. Real-time multi-device / multi-crew sync needs
               the short Firebase setup below (also free).

WHY GITHUB PAGES ALONE DIDN'T UPDATE IN REAL TIME
---------------------------------------------------
GitHub Pages only serves static files — there's no server behind it to hold
a shared "live" copy of the data, so two people opening the page were each
just working with their own browser's local copy. This version fixes that
by adding a tiny, free, real-time backend (Firebase Realtime Database).
Once it's configured below, every open tab is connected to the same live
document and sees changes the instant anyone makes them — no page refresh,
no polling delay, no login screen.

HOW TO HOST THE FILE ITSELF
-----------------------------
Any static host works — GitHub Pages, Netlify, Vercel, Cloudflare Pages, etc.
For GitHub Pages: create a repo, upload index.html, then Settings -> Pages ->
choose the branch/folder, and you'll get a URL like:
    https://yourusername.github.io/your-repo-name/

ENABLING REAL-TIME SYNC (Firebase — free, ~5 minutes, no server code)
-------------------------------------------------------------------------
1. Go to https://console.firebase.google.com -> "Add project" (free tier is
   plenty for this).
2. In the left menu: Build -> Realtime Database -> Create Database.
   Choose any region, then start in TEST MODE.
     - Test mode means anyone with your database URL can read/write it.
       That's intentional here — it's what lets any crew member open the
       link and contribute with no login screen. See the security note
       below if you want to lock it down further.
3. Project settings (the gear icon, top left) -> General tab -> scroll to
   "Your apps" -> click the Web icon (</>) -> register the app (any nickname)
   -> it will show you a firebaseConfig object like:

       const firebaseConfig = {
         apiKey: "AIza...",
         authDomain: "your-project.firebaseapp.com",
         databaseURL: "https://your-project-default-rtdb.firebaseio.com",
         projectId: "your-project"
       };

4. Open index.html in a text editor, find FIREBASE_CONFIG near the top of
   the <script> block, and paste in those four values (apiKey, authDomain,
   databaseURL, projectId).
5. Back in Firebase: Build -> Authentication -> Get started -> Sign-in
   method -> enable "Anonymous". (This just lets each browser tab identify
   itself to the database — nobody sees a login prompt or needs an account.)
6. Re-upload/redeploy index.html to your host.
7. Open the hosted link on two devices — you should see the small "Live
   Sync" status pill at the top turn green, and any take-off / landing /
   maintenance action on one device appears on the other within about a
   second.

SECURITY NOTE ON TEST MODE
----------------------------
Test mode means the database URL, if someone got hold of it, would let them
read or write the board without needing to be one of your crew. For an
internal, non-sensitive flight-line status board this is usually an
acceptable trade-off for the convenience of "no login needed." If you'd
rather restrict who can write:
  - In Firebase, Build -> Realtime Database -> Rules, and require
    `auth != null` (already effectively true since every tab signs in
    anonymously) plus consider Firebase App Check to block requests that
    don't come from your actual hosted page.
  - For real per-person accountability (e.g. only specific Google accounts
    allowed), swap anonymous sign-in for Google sign-in and write rules
    that check the signed-in email — this is a larger change than the
    anonymous approach and happy to help set it up if you want it.

NOTES
-----
- If you skip the Firebase setup, the dashboard still works perfectly on a
  single device — it just keeps everything in that browser's local storage
  only, with no cross-device sync (the "Live Sync" pill will just say "not
  configured").
- The PDF and Excel export buttons load small libraries from a public CDN
  the first time they're used — this needs an internet connection but no
  further setup.
- Nothing here is sent to Anthropic. Data goes only to the Firebase project
  you create and control (or nowhere, if you skip that setup).

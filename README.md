# HomeHunt setup guide

Files in this folder:

| File | What it is |
|---|---|
| index.html | The app both of you use |
| admin.html | Analytics page, only for RP |
| Code.gs | Google Apps Script backend (Sheet, Drive folders, summaries) |
| sw.js, manifest.json, icon-192.png, icon-512.png | Make it installable and partly usable offline |

Time needed: about 25 minutes, once.

## Part 1. Put the app on GitHub Pages

1. Sign in at github.com as sunnygeorge1202-eng.
2. Click **+** (top right), then **New repository**. Name it `homehunt`. Choose **Public** (free GitHub Pages needs public). Tick **Add a README file**. Click **Create repository**.
3. In the repo click **Add file**, then **Upload files**. Drag in `index.html`, `admin.html`, `sw.js`, `manifest.json`, `icon-192.png`, `icon-512.png`. Do NOT upload `Code.gs` or this README (keys will live in Code.gs, and the repo is public). Click **Commit changes**.
4. Go to **Settings**, then **Pages**. Under Source choose **Deploy from a branch**, branch `main`, folder `/ (root)`, click **Save**.
5. Wait 1 to 2 minutes. Your links will be:
   - App: https://sunnygeorge1202-eng.github.io/homehunt/
   - Admin: https://sunnygeorge1202-eng.github.io/homehunt/admin.html

Because the repo is public, anyone who guesses the URL can open the page, but they see no data without your keys. Listing data never goes into GitHub, only into your Google account.

Updating later: open the file in the repo, click the pencil, paste, commit. GitHub Pages caches hard, so if a phone shows the old version, close the app fully and reopen, or add `?v=2` to the address.

## Part 2. Create the Google Sheet and backend

1. Go to sheets.google.com and create a blank sheet named `HomeHunt Tracker`. Copy the long ID in its URL (between `/d/` and `/edit`).
2. In the sheet click **Extensions**, then **Apps Script**. Delete the sample code and paste all of `Code.gs`.
3. At the top set three values:
   - `SHEET_ID`: the ID you copied
   - `SECRET_KEY`: a long random phrase. This one goes on both phones.
   - `ADMIN_KEY`: a different long random phrase. Only you use it, with admin.html.
4. Click **Save**. In the function dropdown choose `setup`, click **Run**, and approve the permissions (Google shows "unverified app", click Advanced, then Go to project; this is your own script). It creates the Drive folders `HomeHunt`, `Listings`, `Archive`, `_diagnostics`, the Sheet tabs, and a 10 minute timer that refreshes summaries. Running it again is safe.
5. Run `doctor`. Open **Execution log**. Every line should say OK. Fix anything it flags before continuing.
6. Click **Deploy**, then **New deployment**, type **Web app**. Set Execute as **Me** and Who has access **Anyone**. Click **Deploy** and copy the **Web app URL** (ends in `/exec`).
7. If you edit Code.gs later: **Deploy**, **Manage deployments**, pencil, Version **New version**, Deploy. The URL stays the same. Without a new version your edits are not live.

## Part 3. Connect the phones

On each phone (hers and yours):

1. Open the app link in Chrome. Menu, then **Install app** (or Add to Home screen).
2. Settings tab, then paste the web app URL and the SECRET_KEY (not the admin key). Enter her name or yours under "You and sharing".
3. Tap Sync. The first time, a consent screen explains that photos and voice notes will be copied to your Google Drive. Both of you should read it. She can choose "Keep on phone only" for any single photo or note.
4. Tap **Test Drive upload**. You should see "Upload works". A test file lands in `HomeHunt/_diagnostics`.
5. Share the Drive folder `HomeHunt` with her Google account as Viewer if she wants to browse it. She does not need to for the app to work.

## Part 4. Admin page

Open admin.html, tap **Connection**, paste the web app URL and the ADMIN_KEY, optionally set the move-in date, and save. It shows days to move-in, a pipeline funnel, daily activity, sources, areas, rent bands, rejection reasons, what worried her on visits, gut feel vs fit score, visits, a needs-attention list, who did what, and per-listing Drive links. Every chart has a Table view. The key stays in that browser only.

## Where things end up in Drive

```
HomeHunt/
  HomeHunt Tracker (Google Sheet: Listings, Events, Media)
  Listings/
    2026-10-12 - BTM Layout - Sunrise Apts - Rs14000 [id]/
      Summary (Google Doc, rebuilt every 10 min when something changed)
      Photos/
      Voice notes/
  Archive/        rejected or deleted listings move here, nothing is erased
  _diagnostics/
```

Text notes live in the Summary doc and the Sheet. Voice notes are .webm audio; Drive may only offer download, not inline play. Open them in Chrome or VLC.

## Edge cases the system handles

- No signal on a visit: photos and notes are stored on the phone and upload later with retry (15 s, doubling, up to 10 min, 8 tries, then marked failed with a Retry button).
- Reply lost after an upload: the app asks the server whether the file arrived, so nothing is uploaded twice.
- Two phones edit the same listing: newest edit wins; the Events tab keeps the history.
- Phone clock wrong: the app corrects using server time and warns if far off.
- Listing renamed, rejected, or its Drive folder trashed: folder is renamed, moved to Archive, or recreated.
- Storage full, private tab, mic blocked, no microphone support: clear messages, nothing lost.
- Wrong or missing key: clear message. The shared key never opens admin.
- Duplicate phone number or link, odd rent, missing phone: warnings before saving.

## Troubleshooting

| Symptom | Fix |
|---|---|
| "wrong key" | Key in app must equal SECRET_KEY exactly; admin needs ADMIN_KEY |
| Admin says URL did not return data | Redeploy as Web app with access Anyone; use the /exec URL |
| Sync works, uploads fail | Run Test Drive upload; redeploy a New version after any Code.gs edit |
| Old app version on phone | Close app fully, reopen; or add ?v=2 |
| Folder missing in Drive | Run `setup` again, then Sync |
| Summary doc not updating | It refreshes every 10 min; run `refreshSummaries` manually to check |
| Mic does not work | Chrome, Site settings, Microphone, Allow |

## Honest limits

- Uploading photos to Apps Script via POST has been tested against a simulated server, not live. Use Test Drive upload first and tell me if it fails.
- The page cannot run in the background; "near the place" prompts work only while the app is open.
- Voice notes are stored, not transcribed.
- Sharing the Apps Script URL plus SECRET_KEY gives access to the data; keep both private.

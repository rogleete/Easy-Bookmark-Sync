# Easy Bookmark Sync 
(open the [INSTALL.md](./INSTALL.md) for first run instructions)

A browser extension for Chrome and Edge that syncs bookmarks between your
own computers through your own Google Drive. One computer is the **Master
Sync Source** (bookmarks get pushed up from here), and any number of other
computers can be a **Destination Sync** (bookmarks get pulled down and
replace whatever's local).

There's no server in the middle - this extension talks directly to Google
Drive using a Google Cloud OAuth Client ID that **you** create yourself.
Nobody's bookmark data or Google account access ever passes through the
extension's developer. See [PrivacyPolicy.md](./PrivacyPolicy.md) for the
full details.

## Features

- Syncs through a single `EasyBookmarkSync` folder created in your Drive
- Sync frequency: Manual only, Realtime, or a fixed interval (10/20/30/45/60 min)
- Realtime mode reacts to bookmark changes within a few seconds on the
  master browser, with a once-a-minute backstop check
- Status and stats shown right in the popup (bookmarks synced / total)
- Manual backups: separate, timestamped snapshots you create on demand,
  stored in their own `Backups` folder untouched by the regular sync -
  browse them, restore one, or delete individual ones any time
- Only asks for the `drive.file` Google scope - it can only see files and
  folders it creates itself, not your whole Drive

## Setup Guide

This extension needs a (free) Google Cloud OAuth Client ID before it can
sign in to Google Drive. It's about ten minutes, one time only, and you
don't need to touch any code - everything gets pasted into the extension's
Options page.

### Part 1: Google Cloud project

1. Go to [console.cloud.google.com](https://console.cloud.google.com/) and
   sign in with the Google account you want to sync bookmarks through.
2. Create a new project (top left project dropdown → New Project). Name it
   whatever you like, e.g. "Bookmark Sync".
3. Search the top bar for **Google Drive API** and click **Enable**.

### Part 2: OAuth consent screen

1. Go to **APIs & Services → OAuth consent screen**.
2. User type: **External** (Internal is fine too if you have a Workspace
   account).
3. Fill in an app name and your email for the support/developer contact
   fields. Everything else can stay blank.
4. Scopes step: nothing to add manually.
5. Test users step: add the Google account email you'll actually use with
   the extension. While the app is in "Testing" mode, only accounts on
   this list can sign in - that's expected for a personal tool.

### Part 3: OAuth Client ID

1. Go to **APIs & Services → Credentials → Create Credentials → OAuth
   client ID**.
2. Application type: **Web application** (not "Chrome Extension" - that
   older client type isn't needed and just adds confusion).
3. Under **Authorized redirect URIs**, add the exact URI shown on the
   extension's Options page (right-click the extension icon → Options).
   It looks like `https://<extension-id>.chromiumapp.org/`.
4. Using this in more than one browser? Add each browser's redirect URI as
   its own line under the same OAuth client - Chrome and Edge generate
   different extension IDs, but one Client ID can hold multiple redirect
   URIs, so you still only need one Client ID total.
5. Save, then copy the Client ID (ends in `.apps.googleusercontent.com`).

### Part 4: Load the extension

**Chrome:** go to `chrome://extensions`, turn on Developer mode, click
**Load unpacked**, select this repo's folder.

**Edge:** go to `edge://extensions`, turn on Developer mode, click **Load
unpacked**, select the same folder.

### Part 5: Paste the Client ID

1. Right-click the toolbar icon → **Options**.
2. Copy the redirect URI shown there, and add it to your OAuth client in
   Google Cloud Console (Part 3, step 3).
3. Paste the Client ID from Part 3 into the field on the Options page and
   click **Save**.
4. Repeat for a second browser if you're using one: open its Options page,
   grab its redirect URI, add it to the same OAuth client, then paste the
   same Client ID in.

### Part 6: First run

Do this on your **master** computer first (the one with the bookmarks you
already have):

1. Click the extension icon.
2. Check **Master Sync Source**.
3. Click **Connect Google Account** and approve access.
4. It creates an `EasyBookmarkSync` folder in Drive and does an initial
   upload.

Then on any other computer you want to pull bookmarks down to:

1. Load the extension there too (repeat Parts 4-5 for that browser if
   needed).
2. Click the extension icon, check **Destination Sync**.
3. Click **Connect Google Account**, sign in with the *same* Google
   account.
4. It pulls down whatever the master last uploaded, replacing local
   bookmarks.

## How syncing behaves

- **Manual**: nothing happens automatically - only the "Sync now" button
  in the popup triggers a sync.
- **Master, Realtime**: a real bookmark change (add/remove/edit/move)
  triggers a sync within a few seconds, plus a once-a-minute backstop
  check in case a change happened while the browser was closed.
- **Destination, Realtime**: checks the cloud roughly once a minute. True
  instant push would need a server watching for changes, so this is the
  closest practical equivalent.
- Every sync fully replaces the target - no merging. Master overwrites the
  cloud copy; destination overwrites local bookmarks.
- Checks that find nothing new don't touch Drive or your bookmarks at all
  - a sync only actually happens when something real changed.

## Manual backups

Separate from the automatic master/destination sync, "Manual Backup" in
the popup footer opens a full page where you can:

- **Generate a separate Backup** - captures every bookmark in this browser
  right now, uploaded as its own timestamped file in a `Backups` folder
  inside `EasyBookmarkSync`, alongside a bookmark count.
- **Browse and delete** - see every backup you've taken, with its date and
  count, and delete individual ones permanently.
- **Restore** - pick a backup from the dropdown and restore it. This
  replaces every current local bookmark with what's in that snapshot and
  can't be undone, so it asks for confirmation first. **Restoring is only
  available on the Master Sync Source browser** - a destination browser's
  bookmarks are just a mirror of the master, so restoring there wouldn't
  do anything the master doesn't already control. If you restore on the
  master, the restored state is also flagged to push up on the next
  regular sync, so it becomes the new live copy instead of getting
  overwritten again.

These backups are completely separate from the live `bookmarks.json` file
the automatic sync uses - creating, restoring, or deleting one never
affects the regular sync, and vice versa.

## Troubleshooting

- **Error 400: redirect_uri_mismatch** - the redirect URI on the Options
  page doesn't exactly match one of the Authorized redirect URIs on your
  OAuth client. Check for a missing trailing slash or http vs https.
- **"No Google Client ID set yet"** - paste one into the extension's
  Options page.
- **Asks to sign in every time** - usually a mismatched redirect URI, or a
  browser profile blocking third-party cookies for accounts.google.com.
- **"No bookmarks found in the cloud yet"** on a destination browser - the
  master browser hasn't completed its first sync yet.

## Privacy

Each person who installs this extension connects it to their own Google
Cloud project - see [PrivacyPolicy.md](./PrivacyPolicy.md) for details on
what data is accessed and where it goes (short version: only to your own
Google Drive, nowhere else).

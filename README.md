# Easy Bookmark Sync - Setup Guide

This extension talks directly to your own Google Drive, there's no server in
the middle. That means before it can work, you need a (free) Google Cloud
OAuth Client ID. About ten minutes, one time only, and no code editing -
everything gets pasted into the extension's Options page once it's loaded.

(Once the extension is loaded, this same guide is also available in-app:
right click the toolbar icon → Options → "View full setup instructions".)

## Part 1: Google Cloud project

1. Go to https://console.cloud.google.com/ and sign in with the Google
   account you want to sync bookmarks through.
2. Create a new project (top left project dropdown → New Project). Name it
   whatever you like, e.g. "Bookmark Sync".
3. Search the top bar for **Google Drive API** and click **Enable**.

## Part 2: OAuth consent screen

1. Go to **APIs & Services → OAuth consent screen**.
2. User type: **External** (Internal is fine too if you have a Workspace
   account).
3. Fill in an app name and your email for the support/developer contact
   fields. Everything else can stay blank.
4. Scopes step: nothing to add manually.
5. Test users step: add the Google account email you'll actually use with
   the extension. While the app is in "Testing" mode, only accounts on this
   list can sign in - normal for a personal tool.

## Part 3: OAuth Client ID

1. Go to **APIs & Services → Credentials → Create Credentials → OAuth
   client ID**.
2. Application type: **Web application** (not "Chrome Extension" - that
   older client type isn't needed).
3. Under **Authorized redirect URIs**, you'll add a URI shown on the
   extension's Options page in Part 5 below - come back to this step after
   loading the extension once. It looks like
   `https://<extension-id>.chromiumapp.org/`.
4. Using this in more than one browser? Add each browser's redirect URI as
   its own line under the *same* OAuth client. Chrome and Edge generate
   different extension IDs even from the same folder, but one Client ID can
   hold multiple redirect URIs, so you still only need one Client ID total.
5. Save, then copy the Client ID (ends in `.apps.googleusercontent.com`).

## Part 4: Load the extension

**Chrome:** go to `chrome://extensions`, turn on Developer mode, click
**Load unpacked**, select the `easy-bookmark-sync` folder.

**Edge:** go to `edge://extensions`, turn on Developer mode, click **Load
unpacked**, select the same folder.

## Part 5: Paste the Client ID

1. Right-click the toolbar icon → **Options** (or open it from the popup's
   setup screen).
2. Copy the redirect URI shown there, and go add it to your OAuth client
   in Google Cloud Console (Part 3, step 3).
3. Paste the Client ID from Part 3 into the field on the Options page and
   click **Save**.
4. Repeat for a second browser if you're using one: open its Options page,
   grab its redirect URI, add that to the same OAuth client, then paste
   the same Client ID in.

## Part 6: First run

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

- **Master, Realtime**: a bookmark change triggers a sync a few seconds
  later, plus a once-a-minute backstop check.
- **Destination, Realtime**: checks the cloud roughly once a minute - true
  instant push needs a server watching for changes, this is the closest
  practical equivalent.
- Every sync fully replaces the target, no merging.
- Only asks for permission to see files it creates in Drive
  (`drive.file` scope), not your whole Drive.

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

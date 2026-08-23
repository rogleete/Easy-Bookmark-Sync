Easy Bookmark Sync - Setup Guide
This extension talks directly to your own Google Drive - there's no server in the middle. That means before it can work, you need a (free) Google Cloud OAuth Client ID. It's about ten minutes, one time only, and you don't need to touch any code - everything gets pasted into the extension's Options page.

Part 1: Google Cloud project
Go to console.cloud.google.com and sign in with the Google account you want to sync bookmarks through.
Create a new project (top left project dropdown → New Project). Name it whatever you like, e.g. "Bookmark Sync".
Search the top bar for Google Drive API and click Enable.
Part 2: OAuth consent screen
Go to APIs & Services → OAuth consent screen.
User type: External (Internal is fine too if you have a Workspace account).
Fill in an app name and your email for the support/developer contact fields. Everything else can stay blank.
On the Scopes step, there's nothing to add manually.
On the Test users step, add the Google account email you'll actually use with the extension. While the app is in "Testing" mode, only accounts on this list can sign in - that's expected for a personal tool.
Part 3: OAuth Client ID
Go to APIs & Services → Credentials → Create Credentials → OAuth client ID.
Application type: Web application (not "Chrome Extension" - that older client type isn't needed and just adds confusion).
Under Authorized redirect URIs, add the exact URI shown on the extension's Options page (right click the extension icon → Options, or the gear/settings link in the popup). It looks like https://<extension-id>.chromiumapp.org/.
Using this in more than one browser? Add each browser's redirect URI as its own line under the same OAuth client - Chrome and Edge generate different extension IDs even from the same folder, but one Client ID can hold multiple redirect URIs, so you still only need one Client ID total.
Save, then copy the Client ID (ends in .apps.googleusercontent.com).
Part 4: Load the extension
Chrome: go to chrome://extensions, turn on Developer mode, click Load unpacked, select the extension folder.

Edge: go to edge://extensions, turn on Developer mode, click Load unpacked, select the same folder.

Part 5: Paste the Client ID
Open the extension's Options page (right-click the toolbar icon → Options).
Paste the Client ID from Part 3 into the field there and click Save.
If you loaded the extension in a second browser, open its Options page too, copy that browser's redirect URI, add it to the same OAuth client back in Google Cloud Console (Part 3, step 4), then paste the same Client ID into that browser's Options page as well.
Part 6: First run
Do this on your master computer first (the one with the bookmarks you already have):

Click the extension icon in the toolbar.
Check Master Sync Source.
Click Connect Google Account and approve access.
It creates an EasyBookmarkSync folder in Drive and does an initial upload.
Then on any other computer you want to pull bookmarks down to:

Load the extension there too (repeat Parts 4-5 for that browser if needed).
Click the extension icon, check Destination Sync.
Click Connect Google Account and sign in with the same Google account.
It pulls down whatever the master last uploaded, replacing local bookmarks.
How syncing behaves
Master, Realtime: a bookmark change triggers a sync a few seconds later, plus a once-a-minute backstop check so nothing gets missed if the browser was closed when the change happened.
Destination, Realtime: checks the cloud roughly once a minute. True instant push would need a server watching for changes, so this is the closest practical equivalent.
Every sync fully replaces the target - no merging. Master overwrites the cloud copy; destination overwrites local bookmarks.
The extension only has permission to see files it creates in Drive (the drive.file scope), not your whole Drive.
Troubleshooting
Error 400: redirect_uri_mismatch - the redirect URI on the Options page doesn't exactly match one of the Authorized redirect URIs on your OAuth client. Double check for a missing trailing slash or http vs https.
"No Google Client ID set yet" - paste one into the extension's Options page.
Asks to sign in every single time - usually a mismatched redirect URI, or a browser profile blocking third-party cookies for accounts.google.com.
"No bookmarks found in the cloud yet" on a destination browser - the master browser hasn't completed its first sync.

---
slug: docs/third-party-access
title: Enable third-party client access
description: Turn on Jellyfin compatibility and give other apps the correct server address.
---

Apps built for Jellyfin can connect to Silo on a separate port, 8096 in the default Docker setup. Jellyfin compatibility is on by default, but until you set an address people can reach, Silo's connection instructions show them `http://127.0.0.1:8096`. Audiobookshelf setup is in the [Beta section](/docs/audiobookshelf), including its profile-PIN warning.

## Set the address for Jellyfin apps

1. Open **Admin > Integrations > Compatibility**.
2. Check that **Allow Jellyfin apps to connect** is on. Changing this switch needs a server restart.
3. In **Address Jellyfin apps should use**, enter the address people should type into their Jellyfin app, for example `http://192.168.1.20:8096` on your home network or `https://jellyfin.example.com` from outside it.
4. Save.
5. Connect one app with a regular account and try browsing and playback before sharing the address.

`localhost` or `127.0.0.1` on a phone, tablet, or TV refers to that device, not your server. If you changed the Docker port mappings, use your own port. The [Docker guide](/docs/docker) lists the default ports.

The **Jellyfin web player** controls install or remove the Jellyfin web player, which Jellyfin mobile and TV apps expect to find on the server. Save the compatibility settings first, then wait for the install to finish.

## Access from outside your network

Give each service its own HTTPS address, for example separate hostnames for Silo and for Jellyfin apps, and point each one at the right port. See the [reverse proxy guide](/docs/reverse-proxy).

Test from outside your home network with the same address you give people. A working Silo web page doesn't mean the Jellyfin port is reachable.

## Accounts and profiles

Give people their Silo credentials and the [Jellyfin sign-in instructions](/docs/jellyfin-apps). Apps that offer Quick Connect sign in with a code the person approves on Silo's `/activate` page, where they choose the profile. Other apps sign in with a combined username and profile, and protected profiles use a combined password and PIN.

Apps differ in which Jellyfin features they use, so note the app and server versions when you report a problem.

## Seerr

Seerr (formerly Jellyseerr) connects to Silo as a Jellyfin server. During Seerr setup, choose Jellyfin and enter the host, port, and HTTPS setting from the address you set in **Address Jellyfin apps should use**.

Seerr shows a user's uploaded Silo profile picture after that user signs in to Seerr with their Silo account. Users with a built-in avatar get Seerr's default picture. If all of a user's Jellyfin sign-ins end, for example after a password change, Seerr shows its default picture until they sign in to a Jellyfin app or Seerr again.

### Relink users after the user ID change

Silo sends Jellyfin user IDs as 32 hex digits without dashes, the way Jellyfin does. Earlier Silo versions added dashes. Seerr recognizes a returning user by that ID, so people who signed in to Seerr before the change no longer match their Seerr account. Depending on Seerr's settings, their sign-in is refused or Seerr creates a second account for them.

Fix this once. If Seerr lets new Jellyfin users sign in, stop it before you update Silo so no one signs in before you've run the fix.

1. If Seerr was running after the update, look for second accounts it created. Run this against Seerr's database:

   ```sql
   SELECT "id", "email", "jellyfinUsername" FROM "user" WHERE "jellyfinUserId" IN (SELECT REPLACE("jellyfinUserId", '-', '') FROM "user" WHERE "jellyfinUserId" LIKE '%-%');
   ```

   Each row is a new account for someone who already has one. Delete those accounts on Seerr's **Users** page.
2. Stop Seerr and back up its database.
3. Run this statement against the database. It also updates the avatar address Seerr saved for each user:

   ```sql
   UPDATE "user" SET "jellyfinUserId" = REPLACE("jellyfinUserId", '-', ''), "avatar" = REPLACE("avatar", "jellyfinUserId", REPLACE("jellyfinUserId", '-', '')) WHERE "jellyfinUserId" LIKE '%-%';
   ```

4. Start Seerr.

Both statements work on Seerr's SQLite and PostgreSQL databases. Jellyfin apps that signed in before the change keep working without this step.

## Legacy Autoscan

An external Autoscan service connects through the Jellyfin port. New setups should use Silo's built-in [Autoscan sources](/docs/autoscan) instead.

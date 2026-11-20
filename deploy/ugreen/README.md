# openGym on a UGREEN NAS

These steps run this fork (password sign-in by default) on a UGREEN NAS with UGOS Pro, such as
the DXP4800 Pro. The images are built by GitHub Actions on the fork and published to
`ghcr.io/redbeard-lab/opengym-api` and `ghcr.io/redbeard-lab/opengym-web` with the tag `edge`.
Every push to `main` that changes the app builds a new `edge`.

## 1. Let the NAS pull the images

New GitHub packages are private. Pick one:

- **Make them public** (simplest; the code is public anyway). On GitHub, open your profile,
  **Packages**, then each of `opengym-api` and `opengym-web`: **Package settings**,
  **Change visibility**, **Public**.
- **Keep them private** and sign the NAS in to the registry. Create a classic personal access
  token with only `read:packages`, then in UGOS Pro **Docker**, **Image**, **Registry**
  (or **Settings**), add `ghcr.io` with your GitHub username and that token.

## 2. Make a folder for the data

In **File Manager**, create `docker/opengym/data` on your volume, so the full path is
`/volume1/docker/opengym/data`. If your volume or share is named differently, use that path in the
compose file instead.

## 3. Create the project

1. Find the NAS's LAN address (UGOS Pro **Control Panel**, **Network**), for example
   `192.168.1.50`. Giving the NAS a fixed address in your router helps, since openGym needs the
   exact address.
2. Open **Docker**, **Project**, **Create**. Name it `opengym` and pick
   `/volume1/docker/opengym` as its folder.
3. Paste [docker-compose.yml](docker-compose.yml) and change `192.168.1.50` to your NAS's
   address in **both** `ORIGIN` and `RP_ID`. If port 3030 is in use, change it in `ports:` and
   in `ORIGIN`.
4. Deploy. Both containers, `opengym-api-1` and `opengym-web-1`, must be running.

## 4. Sign in

Open `http://<NAS address>:3030`, choose **Create new profile**, pick a name and a password.
The first profile becomes the admin. After that, consider uncommenting `INVITE_ONLY=1` so
strangers on your network cannot make profiles, then redeploy.

Passkeys only work over HTTPS. To use them too, put openGym behind the NAS's reverse proxy with a
hostname and certificate, then set `ORIGIN=https://your.hostname` and `RP_ID=your.hostname`
(see `docs/SELF_HOSTING.md`, "On a NAS", step 4).

## Updating

After a new build finishes on GitHub, in **Docker**, **Project**, `opengym`: pull the images
again and redeploy (or **Rebuild**). Your data in `/volume1/docker/opengym/data` stays.

## If something goes wrong

- **"This server only takes changes made at ..."**: `ORIGIN` or `RP_ID` does not match the
  address in the browser. Fix it and redeploy; a plain restart keeps the old values.
- **Sign-in never works, only "Continue without account"**: the `api` container is not running.
  Check its log in **Docker**, **Container**.
- **Image pull denied**: the packages are still private and the NAS is not signed in (step 1).

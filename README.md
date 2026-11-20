# Tokyo Night Drive — Supabase Cloud Saves + Public Multiplayer

This project upgrades the supplied Three.js game with:

- Required Google sign-in through **Supabase Auth** (Firebase is not used)
- Per-account cloud progress in Supabase Postgres, protected by Row Level Security
- A public Colyseus room capped at 8 players
- Server verification of Supabase access tokens before a player can enter the room
- Synchronized player position, heading, car choice, paint color, and floating name tags

## 1. Create the Supabase project

1. Create a project at https://supabase.com/dashboard.
2. In **Project Settings → API**, copy the **Project URL** and the **publishable key** (or legacy `anon` key).
3. Open `public/supabase-config.js` and replace the two placeholders:
   - `url`: your Project URL
   - `anonKey`: your publishable/anon key
4. In the Supabase Dashboard, open **SQL Editor → New query**.
5. Paste all of `supabase/schema.sql` and run it. This creates the `player_saves` table and owner-only RLS policies.

The browser publishable/anon key is intended to be public. The table's RLS policies ensure each signed-in user can only read or change their own save. **Never put a Supabase secret/service-role key in the browser.**

## 2. Enable Google sign-in in Supabase

1. In Supabase, open **Authentication → Sign In / Providers → Google** and enable Google.
2. Create a Google OAuth client for a Web application in Google Auth Platform / Cloud Console.
3. In the Google OAuth client, add your game's deployed origin under **Authorized JavaScript origins** (for example `https://your-game-host.example`).
4. Add the Supabase callback URL shown on the Supabase Google provider page under **Authorized redirect URIs**. It normally looks like `https://YOUR_PROJECT_REF.supabase.co/auth/v1/callback`.
5. Copy the Google Client ID and Client Secret into the Supabase Google provider settings and save.
6. In **Authentication → URL Configuration**, set the Site URL to your game's hosted URL and add that exact URL to the allowed Redirect URLs. For local testing, also add `http://localhost:2567`.

Google OAuth requires the Supabase callback URL at Google and your game's return URL in Supabase. Do not share the Google client secret.

## 3. Configure and run the multiplayer server

The server needs Node.js 20+ and a host that supports a persistent Node process and WebSockets. Set these environment variables on the server host:

- `SUPABASE_URL` — same Project URL used in the browser config
- `SUPABASE_ANON_KEY` — same publishable/legacy anon key (or use `SUPABASE_PUBLISHABLE_KEY` instead)
- `PORT` — the host's supplied port, if applicable

For a local computer with Node.js:

```bash
cd server
npm install
npm start
```

Then open `http://localhost:2567`. Do not open the HTML using a `file://` URL; OAuth needs a hosted origin.

For production, deploy the `server/` Node app on a WebSocket-capable host and set `multiplayerUrl` in `public/supabase-config.js` to the secure server URL, for example `wss://YOUR_SERVER_HOST`. Host the `public/` files at the game's HTTPS URL. The Node server can also serve the public files itself when deployed together.

## 4. Chromebook notes

You do not need to install Firebase or use Linux for the Supabase dashboard steps. You can edit files in a browser-based coding environment if your school permits it. The Colyseus multiplayer server still needs a persistent Node/WebSocket host; a static webpage host alone cannot run it. If your school blocks a required service, ask school IT or use an approved device rather than trying to bypass the restriction.

## Notes

- Public matchmaking uses one room name; Colyseus creates another public room when the current one reaches 8 players.
- The server synchronizes positions and cosmetic car selection. This is an early free-roam sync layer, not an authoritative competitive physics server; each browser still simulates its own vehicle locally.
- A reachable multiplayer server is required for other drivers to appear. Cloud saves are stored in Supabase independently.
- If the browser has an older `hl5` save and the Supabase account has no save yet, the game imports that local save once, then removes the old local copy. Other devices cannot import a save that only existed on a different device.

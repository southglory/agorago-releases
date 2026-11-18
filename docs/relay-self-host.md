# Self-hosting an agorago relay

This guide is generated from the README inside the relay bundle of agorago
v0.13.174. Every release carries its own bundle, `agorago-relay-<version>.tar.gz`,
on the [releases page](https://github.com/southglory/agorago-releases/releases). It builds and
runs without the app's source.

## 0. Get the bundle

Use the bundle from the same release as your app.

```bash
VERSION=v0.13.174
curl -fLO https://github.com/southglory/agorago-releases/releases/download/$VERSION/agorago-relay-$VERSION.tar.gz
curl -fLO https://github.com/southglory/agorago-releases/releases/download/$VERSION/agorago-relay-$VERSION.tar.gz.sha256
sha256sum -c agorago-relay-$VERSION.tar.gz.sha256
tar xzf agorago-relay-$VERSION.tar.gz
cd agorago-relay-$VERSION
```

The steps below run in that folder.

Sharing a terminal on the same network needs nothing extra: pick **Same network**
in agorago's share popover. Opening it from anywhere else (a phone on mobile
data, another office) needs a relay both ends can reach. agorago ships no
built-in relay and no default address: you run this one on your own server and
register it in the app.

This folder is everything the relay needs: `server.mjs` (one small Node process,
a WebSocket server built on `ws`), the operator page `admin.html`, and the viewer
page under `viewer/`, built for the agorago version this bundle came with. It
keeps no account and no database. The only files it writes are the people you
approved, the passes you issued and the invites hosts created, under `/data`.

Use the bundle from the same release as your app. A newer app works with an
older relay, but features the relay does not offer yet (such as refusal reasons
and removing a viewer) stay off.

## What you need

- A server with a public HTTPS name, for example `relay.example.com`.
- Docker with Compose 2.24 or newer, or Node 22 or newer.
- A reverse proxy that terminates TLS. The relay speaks plain HTTP and WS on
  port 8080; the app and the browser viewer need `https://` and `wss://`.

## 1. Configure

```bash
cp env.example .env
sed -i "s|^RELAY_ADMIN_SECRET=.*|RELAY_ADMIN_SECRET=$(openssl rand -base64 32)|" .env
chmod 600 .env
```

| Variable | What it does |
|---|---|
| `RELAY_ADMIN_SECRET` | Lets you approve requests, issue passes and revoke them. Setting it also locks the relay: opening a room then needs a pass. Without it, anyone who finds the address can open rooms. |
| `RELAY_TRUSTED_PROXIES` | How many proxies sit in front. `1` behind one Caddy or nginx, so login throttling counts real client addresses. |
| `RELAY_VIEWER_BASE` | The public viewer address, if not `https://<your host>/viewer`. |
| `RELAY_HOST_SECRETS` | `name:secret` pairs, comma separated, for holders you add by hand. |
| `RELAY_REQUESTS` | `0` hides the in-app access request, when you hand out passes yourself. |
| `RELAY_INVITES_ONLY` | `1` refuses viewers holding only the room link, so only invited people can watch. |
| `RELAY_PORT` | Listen port, default `8080`. With Compose, change the host side of `ports:` instead. |
| `RELAY_MAX_VIEWERS`, `RELAY_MAX_ROOMS`, `RELAY_PASS_ROOMS` | Limits: viewers per room (16), rooms (1000), devices per pass (3). |

## 2. Run

With Docker:

```bash
docker compose up -d --build
docker compose logs relay      # there should be no "could not persist"
```

It listens on `127.0.0.1:8080` only, for the proxy on the same machine.

Without Docker:

```bash
npm ci --omit=dev
set -a; . ./.env; set +a
RELAY_GRANTS_PATH=./data/grants.json \
RELAY_PASSES_PATH=./data/passes.json \
RELAY_INVITES_PATH=./data/invites.json \
node server.mjs
```

The three `RELAY_*_PATH` lines keep state in `./data`; without them the relay
tries `/data` (the container's volume) and logs "could not persist". This
listens on all interfaces: firewall the port so only the proxy reaches it.

To keep it running after you log out, run it as a service, for example with
systemd (`/etc/systemd/system/agorago-relay.service`):

```ini
[Service]
WorkingDirectory=/opt/agorago-relay
EnvironmentFile=/opt/agorago-relay/.env
Environment=RELAY_GRANTS_PATH=/opt/agorago-relay/data/grants.json
Environment=RELAY_PASSES_PATH=/opt/agorago-relay/data/passes.json
Environment=RELAY_INVITES_PATH=/opt/agorago-relay/data/invites.json
ExecStart=/usr/bin/node server.mjs
Restart=on-failure
User=agorago-relay

[Install]
WantedBy=multi-user.target
```

## 3. TLS

Send everything to the relay; it serves the WebSocket at `/rtc`, the viewer at
`/viewer` and its own API. With Caddy:

```caddyfile
relay.example.com {
	reverse_proxy 127.0.0.1:8080
}
```

Check from outside, the way a user reaches it:

```bash
curl https://relay.example.com/healthz          # ok
node check-routes.mjs https://relay.example.com
```

A 400 or 401 means the relay answered; a 404 means the proxy did not carry the
request.

## 4. Point agorago at it

1. **Settings, Web sharing**, **Add server**: enter `wss://relay.example.com/rtc`
   and leave the viewer address blank; the app asks the relay.
2. **Manage** on the server row: **Server check** shows the version, viewer
   address, access control and features. Fix anything marked there.
3. **Open operator tools**, enter `RELAY_ADMIN_SECRET`, **Issue pass** for
   yourself, paste the invite it copies into **Pass**, **Register this device**.
4. In the share popover choose **Anywhere**. The link it copies opens the viewer
   on your relay.

The operator tools are also at `https://relay.example.com/admin` in a browser.

## 5. Let others in

Give people the relay address. They can **Request access** from the share
popover (you approve it in the operator tools), or you **Issue pass** and send
them the invite it copies, the way you would send a password. Each person holds
a separate credential, so **Revoke** locks out one person only.

## What the relay can see

In **Relay** mode, the default, terminal output passes through your server in
the clear. Switching a share to **P2P** sends it over a direct, encrypted WebRTC
connection and the relay sees only the handshake (P2P can fail on strict
networks and then falls back to the relay). Tell the people who use your relay
that it is yours, and that the switch exists. The relay logs events only (room
opened, access granted or revoked) and never stores what passes through.

## Updating

Replace this folder with the bundle from the new release, keep your `.env`, and
run `docker compose up -d --build` again. The data volume keeps approvals and
passes.

## License

Apache License 2.0. See `LICENSE` and `NOTICE`.

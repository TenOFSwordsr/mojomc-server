# MojoMC - a Minecraft 1.21.8 server written in pure Mojo

Native binary, no Java, no Python runtime. Listens on port 25565,
speaks the real Minecraft wire protocol (protocol 772), offline mode
(no Mojang auth), running inside WSL Ubuntu-24.04 on this machine.

## What works now

- Server-list ping (Minecraft multiplayer screen shows "Mojo native server")
- Full offline login: any username, no password
- Configuration handshake: feature flags, brand channel, finish/ack
- Join sequence: join game, creative abilities, spawn position, position sync
- Keepalive echo in play state

## How to run

Open a WSL shell (Ubuntu-24.04) - or any terminal with `wsl` access:

    mojomc-start     # start the server in the background
    mojomc-log       # watch the live log
    mojomc-status    # is it running?
    mojomc-stop      # stop it

Then in Minecraft Java 1.21.7/1.21.8: Multiplayer -> Add Server ->
address `localhost`.

## Honest status: what a real client will see

The protocol flow is verified byte-for-byte with a Python test client
(`logintest.sh`). A REAL Minecraft client should reach the login/config
stages cleanly. It will likely NOT render a world yet: the server sends
the join sequence but no chunk data, and 1.21.8 clients require chunks
(chunk data packets with valid paletted containers + light) before
showing terrain - until then you may see "Loading terrain..." forever
or an immediate disconnect.

Registry data is intentionally skipped: the client announces it knows
the `minecraft:core` pack in configuration state, and vanilla servers
skip registry_data in that case. This is the same optimization real
servers use.

## Files

- `server.mojo`     - the full server source (single file)
- `mojomc`          - compiled binary (also at /usr/local/bin/mojomc in WSL)
- `mojomc-start.sh` - the launcher (also at /usr/local/bin/mojomc-start)
- `logintest.sh`    - Python test client that walks the full login flow
- `prototest.sh`    - status-ping test client
- `mcstatus.mojo`   - earlier status-only version (kept for reference)

## Rebuild after editing server.mojo

    cd /mnt/c/Users/Administrator/Documents/Projects/mojo-jvm
    /root/.pixi/bin/mojo build server.mojo -o mojomc
    cp mojomc /usr/local/bin/mojomc
    mojomc-stop; mojomc-start

## Next steps (in order)

1. Send chunk data in play state (empty superflat columns) - needed for world render
2. Respond to client settings / held-item / position packets properly
3. Place a bedrock floor at y=-64 so the player doesn't fall into the void

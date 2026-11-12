# Velvet server adapter

Runs [Velvet](https://github.com/aroundmyroom/Velvet)'s own Sendspin server
(`src/sendspin/`: `SendspinServer` and `RoomManager`) as a server row of the
matrix. Velvet implements the server role only.

- `server.mjs`: the CLI the harness launches. It starts `SendspinServer` on
  `127.0.0.1:<--port>/sendspin` with mDNS off, registers its URL in the harness
  registry (client-initiated) or dials the client's registered listener
  (server-initiated), puts the client in a room and drives the scenario through
  `RoomManager`. Player scenarios play the first `--clip-seconds` of the
  fixture, cut as the harness cuts it; metadata plays the fixture from
  `--metadata-track-progress`; controller sets repeat and shuffle and waits for
  the client's command; artwork queues the track so the room sends its image.
- `trace.mjs`: the evidence. Velvet has no trace hook, so the adapter wraps
  Velvet's Noise `CipherState` and the `ws` socket from outside and records
  every message as plaintext (outbound right before encryption, inbound right
  after decryption) and each outbound write once `ws` reports it flushed. All
  summary fields are read back from that trace, never from adapter arguments.

Identity: the server key and the long-term pairing record for the harness
client are derived from `--server-id` and `--client-id` the same way the
aiosendspin adapters derive theirs, so pre-paired clients take the `lt` path.
Clients with their own identity connect with the Sentinel PSK and are approved
for unpaired access, as in the aiosendspin server adapter. All harness peers
are loopback, so the per-address admission caps are raised.

## Build

`velvet-server` in `src/conformance/build.py` runs `npm ci --omit=dev` in
`repos/velvet` and `npm install` here. Velvet downloads its own ffmpeg on first
boot, which a bare checkout has not done, so the adapter uses the
`ffmpeg-static` and `ffprobe-static` binaries installed here. `VELVET_FFMPEG`
and `VELVET_FFPROBE` override them, and `CONFORMANCE_REPO_VELVET` points at
another Velvet checkout.

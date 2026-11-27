[![Peer-to-Peer - GameSpice addon](docs/banner.png)](https://moddingcommunity.com/a/77-gs-peer-to-peer)

Sessions that players host for each other: join codes, a lobby, readiness, deterministic host migration, and a plain account of what NAT will not let you do.

Part of the [**GameSpice**](https://gamespice.net) collection: modular, MIT-licensed building blocks for games on the GameSpice platform (only the TMC website back end is closed). The addon is MIT, built with Claude Code, and ships a headless suite (`p2p_selftest`, see Test). It is used for real: the games in [Zee's Games](https://zeesgames.com) are built on GameSpice addons and run on live servers.

## Install

Copy `addons/gs_peer_to_peer/` and [GameSpice Core](https://github.com/gamespice-platform/core)'s `addons/gs_core/` into your project and enable gs-peer-to-peer in **Project > Project Settings > Plugins**. gs-core is the only dependency. WebRTC is optional and is reached by name at runtime.

## Use

```gdscript
var p2p := GSP2PSession.new()
p2p.config = my_config
p2p.signaller = GSP2PSignallerHttp.new(url, my_id, http)
add_child(p2p)
p2p.setup()

var code = (await p2p.host("Ada")).value   # show this to a friend
await p2p.join("K7M4PX", "Bob")                # or type theirs
```

A `GSP2PSignaller` is four calls (verbs). A loopback one ships for tests and split screen, and an HTTP one ships for anything with a URL and four routes. A lobby service or an existing WebSocket is a subclass.

If WebRTC is missing (a desktop build without the GDExtension) the code still parses and runs:

```gdscript
if not GSP2PSession.available():
    push_warning(GSP2PSession.unavailable_reason())
```

## How it works

| Part | What it does |
| --- | --- |
| `GSP2PLobby` | Members, readiness and join slots. It holds no connection, so a headless test covers every race (two people taking one slot, a reused code, two hosts). |
| Host election | `lobby.elect_host()` is a pure function: stable first, then longest in the session, then lowest id. Every peer computes it from the member list it already has, so no messages are needed when a host disappears. |
| Join code | Six characters from `23456789ABCDEFGHJKLMNPQRSTUVWXYZ` (no 0, O, 1, I or l), about a billion codes. Its shape is checked before any lookup. |
| `GSP2PConfig.Trust` | How far a host is trusted: host-authoritative (co-op among friends), verified (catches a careless host, not a determined one) or sandboxed (no records, statistics or leaderboard leave the session). |

## Limits you cannot code around

- **Something must introduce the players.** Two machines behind two routers need a server both can reach. P2P only removes the server from the game traffic.
- **Some pairs cannot connect without a relay.** Roughly 5 to 15 per cent (symmetric or carrier-grade NAT, some corporate and mobile networks). `relay_servers` is empty by default because a relay costs bandwidth. With none set, those pairs fail quickly with a clear reason.
- **A host is a player and can cheat.** Pick the trust level on purpose, and do not feed a leaderboard from a host you do not verify.

## Test

```bash
godot --headless --path . --import
godot --headless --path . res://examples/p2p_selftest.tscn
```

## Used by

These games carry it in their addons folder:

- [Hungario](https://zeesgames.com/hungario)
- [Airdrift](https://zeesgames.com/airdrift)
- [Zombie Zone](https://zeesgames.com/zombie-zone)
- [Fatal Contact](https://zeesgames.com/fatal-contact)
- [Playlot](https://zeesgames.com/playlot)

The whole collection is at [zeesgames.com](https://zeesgames.com).

## Licence

MIT. See [LICENSE](LICENSE).

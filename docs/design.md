# Crash Team Racing Nitro-Fueled on Nintendo Switch — network stack and feasibility

Status: **research verdict, 2026-09-09. No implementation this round.**
Author: tobagin. Clean room: no earlier source was read for this document; see
[Provenance](#provenance).

## Verdict, in one paragraph

CTR:NF on Switch is **not a NEX title**. It authenticates against **Demonware**,
Activision's own backend, over TLS on port 443 to
`lavender-switch-auth3.prod.demonware.net` and
`lavender-switch-lobby.prod.demonware.net`. Nintendo's NEX, NPLN and BAAS carry
none of its online traffic. So this repository is the right shape and it must
**not** be folded into [`servers/nex`](../../nex).

Feasibility is **plausible but unproven, and possibly unnecessary**. Unlike
[`fall-guys`](../../fall-guys), there is no authoritative server simulation to
reimplement — racing is peer-to-peer, so the missing piece is auth plus
matchmaking, which is a network service and squarely what OpenPak does. There is
even AGPL-3.0 prior art for the protocol family. But nobody has yet seen a single
payload byte from this title, and one cheaper question comes first: **does an
OpenPak console need us for this title at all?** Answering that is one afternoon
of work and it can close the repository permanently. It has not been answered, so
no code ships this round.

## 1. Which stack — Demonware, first-hand

### The evidence

| Fact | Source | Strength |
|---|---|---|
| The client resolves `lavender-switch-auth3.prod.demonware.net` and `lavender-switch-lobby.prod.demonware.net` | Our own citron tree: `emulators/citron/src/core/hle/service/sockets/sfdnsres.cpp:206-230` — an exact-host redirect hook whose comment records "confirmed live 2026-08-31" | **Decisive.** Observed console behaviour, the strongest input the [clean-room policy](../../../docs/clean-room-policy.md) admits |
| Both hosts are dialled on **443** | `emulators/citron/src/core/hle/service/sockets/bsd.cpp:1908-1930` — the companion port-remap block rewrites destination port 443 | Decisive, same origin |
| The client's own symbols include `igNetTaskLsgGetTicket` and `igNetTaskLsgAuthenticateTicket` | Static analysis of the title's binary, recorded in the same hook comment | Strong |
| Nintendo's documented Switch NEX game servers do not include this title | [kinnay/NintendoClients wiki][kinnay] lists first-party servers only | Weak (absence of evidence), consistent |
| Online play is peer-to-peer, with skill-based matchmaking added Oct 2019 | [Bandipedia][bandi] ("requires a stable peer-to-peer internet connection"); [Nintendo Life][nl] on SBMM; [Activision support][acti] describes Matchmaking and Private Match | Public, consistent |

`auth3` and `LSG` (Lobby Service Gateway) are the Demonware **BitDemon** family's
own names, and `igNet*` is the IG engine's network layer. Three independent
signals — the hostnames, the port, the symbol names — all say the same thing.

### What this rules out

Nintendo NEX is not involved. NEX titles reach `*.nintendo.net` game servers
selected by a game server ID and are covered by the generic Nintendo redirect in
our emulators; this title's hosts are not, which is exactly why a title-specific
hook had to be added. Nothing here belongs in the NEX title fleet.

### What Nintendo still supplies

Only identity. The console signs in to Nintendo (BAAS/NSA) and Nintendo Switch
Online membership gates online play. The token that sign-in produces is
presumably what the client presents to `auth3`. That single dependency is the
whole question in section 2.

## 2. Rung one — does this need to exist at all?

OpenPak redirects **Nintendo** hostnames. It does not redirect
`*.prod.demonware.net`. An OpenPak console therefore still reaches the real,
still-running Demonware over the open internet. The title breaks on an OpenPak
console only if Demonware refuses the Nintendo credential we mint.

Three outcomes, and we do not know which is ours:

1. **auth3 ignores the Nintendo token** (treats the console as an anonymous
   title-scoped device). CTR:NF online already works on an OpenPak console with
   zero server work → **close this repository**, document "do not redirect
   Demonware" and move on.
2. **auth3 validates the token upstream with Nintendo.** It rejects ours, the
   same wall [`outbound`](../../outbound) hit with Photon's `NintendoSwitch`
   custom-auth provider and [`fall-guys`](../../fall-guys) hit with EOS Connect
   → a replacement is the only way this title works on OpenPak, and section 3
   applies.
3. **The service is shut down** (it is not, as of 2026-09) → preservation case,
   section 3 applies with more urgency.

This is [E1](evidence-needed.md#e1--does-real-demonware-accept-an-openpak-console)
and it is the cheapest experiment in the file. Nothing else should be built
before it answers.

## 3. If a replacement is needed — the scoped plan

### What has to be replaced

| Layer | What it does | Replaceable? |
|---|---|---|
| `auth3` | Issues a ticket for a platform credential | Yes — we mint the ticket, we set the rules |
| `LSG` + lobby services | Ticket exchange, then profile, storage, stats, presence | Yes — protocol family has public prior art |
| Matchmaking / sessions | Puts eight players in one race | Yes in principle, **no public prior art**, see U3 |
| The race itself | Physics, items, positions | **Not ours.** Peer-to-peer between the consoles. We hand out addresses; they play |

That last row is the reason this is tractable and Fall Guys is not.

### Prior art we may use

[`Laupetin/open-bitdemon-emulator`][obd] — **AGPL-3.0**, Rust, and therefore
licence-compatible with OpenPak as a reference (we would reimplement in Go, per
house rules). It splits exactly the way the observed hosts do: `libbitdemon`
with an auth server (`auth/auth_server.rs`, `auth/result/auth_ticket.rs`,
`auth/key_store.rs`) and a lobby side entered through `lobby/lsg.rs`, plus
services for profile, storage, content streaming, counters, groups, rich
presence, leagues and anti-cheat. [`demonware-companion`][dwc] documents the
reverse-engineering method and tooling.

Two caveats, both material:

- It targets **older** games on plain TCP. Ours is TLS on 443. The framing
  inside the tunnel may be BitDemon, or a newer HTTPS layer wearing the same
  service names (U1).
- Its service list has **no matchmaking and no sessions**. The part CTR:NF needs
  to actually race is the part nobody has published (U3).

### Phases, gated

- **P0 — evidence.** E1 through E3 in [`evidence-needed.md`](evidence-needed.md).
  No Go until E2 has shown what the framing is.
- **P1 — auth3 + LSG.** One Go process, self-signed TLS, both hosts pointed at
  it, until the in-game Online menu stops erroring. Success is a screen, not a
  test suite.
- **P2 — matchmaking.** Enough session bookkeeping for two consoles to find each
  other and be told each other's addresses; then get out of the way and let the
  peers race. NAT assistance reuses [`nat-check`](../../nat-check) if it is
  needed at all.
- **P3 — persistence.** Profile, storage and stats backed by
  [`account`](../../../account) through the nx-baas internal game API on 20070,
  the same way every other OpenPak title service does it.

Non-goals, forever unless someone asks: leaderboard parity, Grand Prix live
content, the Pit Stop economy, anti-cheat, telemetry.

### Ports

**No block is claimed.** The 22600–22699 third-party range in
[`ports.md`](../../../ports.md) has 22600–22609 taken by
[`among-us`](../../among-us); this project takes the next free ten in the PR that
first ships a listener, and not before.

## 4. Unknowns

Written as unknowns on purpose. Each maps to an experiment.

- **U1 — what is inside the TLS.** BitDemon framing, or an HTTPS/protobuf layer?
  *Unknown, needs a capture of the first client bytes on a TLS-terminating stub
  (E2).*
- **U2 — what auth3 wants.** Which credential, which per-title key, whether the
  ticket is encrypted with a key baked into the client. *Unknown, needs E3.* Any
  such key material stays off this repository, permanently.
- **U3 — the matchmaking protocol.** No public prior art. *Unknown, needs E5, and
  E5 is expensive.*
- **U4 — does the client hard-fail before the network.** Fall Guys' EOS SDK
  aborts client-side no matter what the server answers; if CTR:NF does the same
  the project dies regardless of the protocol. *Unknown, falls out of E2.*
- **U5 — hardware parity.** Everything above is emulator-side. A real console
  needs the Demonware names in its Atmosphere hosts file and our certificate
  trusted. *Unknown, needs E6.*

## 5. Reopen / close conditions

- **Close permanently** if E1 shows real Demonware accepts an OpenPak console.
  The right production behaviour is then "do not redirect `*.demonware.net`",
  same conclusion [`fall-guys`](../../fall-guys) reached for `api.epicgames.dev`.
- **Proceed to P1** if E1 shows rejection and E2 shows a framing we can speak.
- **Close** if E2 shows the client aborts locally (U4) or the payloads are
  opaque under a key we will not extract.

## Provenance

- Public sources are linked inline.
- First-hand facts come from **OpenPak's own tree**: the citron redirect hooks in
  `emulators/citron` cited above. Those hooks record a live observation and a
  static-analysis result; their original evidence log lives in a
  source-available-Shield-licensed repository which **was not opened** for this document
  and must not be. E2 re-derives the same facts from our own capture, which is
  the only version of them this project is allowed to build on.
- No earlier source, layout, type names or handler names were read or mirrored.

[kinnay]: https://github.com/kinnay/NintendoClients/wiki
[bandi]: https://crashbandicoot.fandom.com/wiki/Crash_Team_Racing_Nitro-Fueled
[nl]: https://www.nintendolife.com/news/2019/10/beenox_adds_skill-based_online_matchmaking_to_crash_team_racing_nitro-fueled
[acti]: https://support.activision.com/crash-team-racing/articles/crash-team-racing-nitro-fueled-online
[obd]: https://github.com/Laupetin/open-bitdemon-emulator
[dwc]: https://github.com/hosseinpourziyaie/demonware-companion

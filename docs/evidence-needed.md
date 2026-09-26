# Evidence needed

[`design.md`](design.md) says CTR:NF is a Demonware title (settled) whose
replacement is plausible, unproven, and possibly unnecessary. This is the
ordered list that decides it. **Cheapest and most decisive first — E1 can close
the repository outright, so do not run E2 before E1.**

Read `https://github.com/openpak/docs/blob/main/playbooks/online-compatibility-playbook.md` (layered model,
experiment discipline) and `emulator-launch-protocol.md` before touching an
emulator. In particular:

- One variable per run; a failed experiment is a result.
- Dumps, captures, Ghidra projects and logs stay **out of this repository** and
  off `/tmp` (tmpfs — filling it kills the shell tool). Disk-backed scratch on
  the pool only.
- Other titles share this machine. Never broad-`pkill`; identify instances by
  full binary path and by their `OPENPAK_*` environment.

## Tooling that already exists

In `Openpak/emulators/citron`, all default-off:

| Control | Effect |
|---|---|
| `OPENPAK_CTR_AUTH_IP=<ip>` | Redirects `lavender-switch-auth3.prod.demonware.net` and `lavender-switch-lobby.prod.demonware.net` (exact host, both resolution paths) |
| `OPENPAK_CTR_AUTH_PORT=<port>` | Moves the destination off 443 when 443 is taken locally (`ss -ltnp`) |
| `OPENPAK_ENABLE=1` + `OPENPAK_SERVER_IP` | The Nintendo-side redirect — identity only; it does **not** touch Demonware |

TLS verification is bypassed in citron's OpenSSL backend, so a self-signed
certificate generated in memory is enough for any stub. Log:
`~/.local/share/citron/log/citron_log.txt`, truncated on every start. Per-title
one instance per data dir: <https://github.com/openpak/docs/blob/main/playbooks/online-compatibility-playbook.md> §0.

## E0 — get the title

**Blocking. Nothing below can run without it.** There is no CTR:NF dump on this
machine (checked 2026-09-09: no NSP/XCI, no save under `switch backup/JKSV`, no
citron entry). Needed: a dump of the user's own cartridge, base plus the active
update, kept on the pool, never in a repository.

Until E0, this whole file is unrunnable and the repository stays as it is.

## E1 — does real Demonware accept an OpenPak console?

**The experiment that decides whether this project exists.** No redirect, no
stub, no server work.

- **Method.** Boot the title in citron with the OpenPak identity stack only —
  `OPENPAK_ENABLE=1`, `OPENPAK_SERVER_IP=<our BAAS>`, `OPENPAK_CTR_AUTH_IP`
  **unset**, so Nintendo names go to us and Demonware names go to production.
  Sign in, enter Online, and watch `citron_log.txt` for the connections to the
  two `prod.demonware.net` hosts.
- **Decision rule.**
  - Online works, or at least authenticates → **close the repository.** The
    correct production behaviour is to leave `*.demonware.net` unredirected, and
    the README says so. Record the run.
  - Authentication is refused → the design's outcome 2 holds; continue to E2.
  - Nothing dials at all → the failure is upstream of the network, in Nintendo
    sign-in or the NSO membership gate; fix that first, it is not this
    project's problem.
- **Care.** This run touches a live third-party service with a non-genuine
  credential. One run, one account, no retry loops, no automation against it.

## E2 — what is inside the TLS?

Answers U1 and U4.

- **Method.** A throwaway TLS-terminating stub on a spare port (self-signed,
  in-memory key, never written to disk), `OPENPAK_CTR_AUTH_IP=127.0.0.1`,
  `OPENPAK_CTR_AUTH_PORT=<port>`. Accept, log the first bytes the client sends
  as hex, answer nothing. Do the same for the lobby host in a second run.
- **Decision rule.**
  - Bytes look like HTTP (`GET`/`POST`, or an HTTP/2 preface) → it is a modern
    HTTPS service; the BitDemon prior art is naming only, and the plan is a
    protocol dig, not a port of anything.
  - Bytes look like length-prefixed binary → likely BitDemon framing; compare
    against [`open-bitdemon-emulator`][obd]'s message layout and continue.
  - The client aborts before or during the handshake → **U4 is fatal**, record
    it and stop; the gate is client-local and this becomes a patching project,
    not a server project.
- **Discipline.** Log to the pool. Never commit a capture.

## E3 — what does auth3 want?

Answers U2. Only after E2 shows a framing we can read.

- **Method.** Extend the stub to parse one request. Identify what credential is
  presented (a Nintendo NSA/BAAS token? a device id? a title-scoped id?) and
  whether the request or the expected reply is encrypted under a key the client
  carries.
- **Decision rule.** A parseable request with an answerable reply shape →
  proceed to P1. An opaque blob under an embedded per-title key → stop and
  reconsider; we can extract such a key for our own analysis but it must never
  enter this repository or any deployment, and a design that depends on shipping
  it is not a design we can ship.

## E4 — does the client accept our ticket?

The first success milestone, and the first line of Go.

- **Method.** Minimal Go `auth3` answering with a ticket of our own, then LSG
  answering `GetTicket`/`AuthenticateTicket` (the client's own symbol names,
  from static analysis).
- **Decision rule.** The Online menu loading without an error is the pass. Note
  every service the client calls next — that list is P3's scope, and it is
  currently guesswork.

## E5 — the matchmaking protocol

The expensive one. Answers U3, and there is no public prior art to lean on.

- **Method.** With E4 passing, drive two isolated citron profiles into
  matchmaking and record the full service call sequence. Expect the answer to
  include peer addresses, because the race itself is peer-to-peer.
- **Decision rule.** A session flow we can reproduce → P2 is real work with a
  known shape. A flow that depends on a service we cannot answer → document the
  wall and stop at "auth works, racing does not".

## E6 — hardware parity

Last. Only once two emulator clients race.

- **Method.** On a real console on the OpenPak rig, add the two Demonware
  hostnames to the Atmosphere hosts file pointing at our server, with the CA
  patch already in place, and repeat E4.
- **Decision rule.** Console behaviour is the only behaviour that counts.
  Expect hardware-only bugs; that has been true every time so far.

[obd]: https://github.com/Laupetin/open-bitdemon-emulator

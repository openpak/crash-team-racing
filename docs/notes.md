# Notes: crash-team-racing

> **Current Switch client: Harbor**, at
> `~/REPOS/Openpak/emulators/harbor`. Harbor is an independent
> OpenPak project based on Eden. Citron, Ryujinx (desktop and Android), and
> the former OpenPak Eden checkouts have been removed. References to those
> forks below record historical evidence only: do not run their commands,
> reinstall them, or select them for current work. Use Harbor's README and
> current handoff for builds and testing; old results do not establish Harbor parity.

Feasibility study for replacing the network **Crash Team Racing Nitro-Fueled**
uses on Nintendo Switch.

## Background

- Racing is peer-to-peer, so there is no server-side game simulation to
  reimplement. That makes this far more tractable than
  [`fall-guys`](../../fall-guys) — the missing piece is auth plus matchmaking, which
  is an ordinary network service.
- The protocol family has AGPL-3.0 prior art
  ([`open-bitdemon-emulator`](https://github.com/Laupetin/open-bitdemon-emulator)),
  licence-compatible as a reference.
- But nobody has seen a payload byte from this title, its transport is TLS on
  443 rather than the plain TCP that prior art targets, and matchmaking — the
  part the game actually needs to race — has no public prior art at all.

- [`docs/design.md`](design.md) — the stack, the evidence, the verdict, and
  the plan that applies only if the replacement turns out to be needed.
- [`docs/evidence-needed.md`](evidence-needed.md) — the experiments in
  order, cheapest first, with the tooling already present in
  `Openpak/emulators/citron`.

## Ground rules

- Clean room. Read [clean-room policy](https://docs.openpak.org/contributors/clean-room/)
  before writing a line. Public protocol documentation, our own captures, and
  compatibly licensed code are the only inputs. Provenance for this study is
  recorded at the end of `docs/design.md`.
- AGPL-3.0-only, English only, Go 1.25, no secrets in the repository — and no
  extracted key material, ever, however convenient.

This project is not affiliated with or endorsed by Nintendo, Activision, Beenox,
Demonware, or any original service provider. It contains no proprietary game
code, assets, SDK material, binaries, certificates, keys, firmware, extracted
files, credentials, or raw packet captures. Crash Team Racing Nitro-Fueled is a
trademark of Activision Publishing, Inc.

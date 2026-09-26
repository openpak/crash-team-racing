# crash-team-racing

Feasibility study for replacing the network **Crash Team Racing Nitro-Fueled**
uses on Nintendo Switch.

## Status: documentation only. No code, and none until the first experiment runs.

**It is a Demonware title, not a NEX title.** The Switch client authenticates
against Activision's own backend over TLS on port 443 —
`lavender-switch-auth3.prod.demonware.net` and
`lavender-switch-lobby.prod.demonware.net`, observed live in our own emulator
tree. Nintendo's NEX, NPLN and BAAS carry none of its online traffic, so this
does **not** belong in [`servers/nex`](../nex) and this repository is the right
shape.

Whether it should hold any code is a different question, and the honest answer
today is *we do not know yet*:

- Racing is peer-to-peer, so there is no server-side game simulation to
  reimplement. That makes this far more tractable than
  [`fall-guys`](../fall-guys) — the missing piece is auth plus matchmaking, which
  is an ordinary network service.
- The protocol family has AGPL-3.0 prior art
  ([`open-bitdemon-emulator`](https://github.com/Laupetin/open-bitdemon-emulator)),
  licence-compatible as a reference.
- But nobody has seen a payload byte from this title, its transport is TLS on
  443 rather than the plain TCP that prior art targets, and matchmaking — the
  part the game actually needs to race — has no public prior art at all.

And one cheaper question comes before all of it. OpenPak redirects *Nintendo*
hostnames; it does not redirect `*.demonware.net`. An OpenPak console still
reaches the real, still-running Demonware. **If Demonware accepts it, this title
already works and this repository should close.** That experiment
([E1](docs/evidence-needed.md#e1--does-real-demonware-accept-an-openpak-console))
is one afternoon and it is unrun.

So: no Go, no port block, no deploy target, until E1 answers.

- [`docs/design.md`](docs/design.md) — the stack, the evidence, the verdict, and
  the plan that applies only if the replacement turns out to be needed.
- [`docs/evidence-needed.md`](docs/evidence-needed.md) — the experiments in
  order, cheapest first, with the tooling already present in
  `Openpak/emulators/citron`. Blocked on E0: there is no dump of this title on
  this machine.

## Ground rules

- Clean room. Read [`docs/clean-room-policy.md`](https://github.com/openpak/docs/blob/main/clean-room-policy.md)
  before writing a line. Public protocol documentation, our own captures, and
  compatibly licensed code are the only inputs. Provenance for this study is
  recorded at the end of `docs/design.md`.
- AGPL-3.0-only, English only, Go 1.25, no secrets in the repository — and no
  extracted key material, ever, however convenient.
- Ports come from [`ports.md`](../../ports.md). Claim a block in the PR that
  first needs one.

This project is not affiliated with or endorsed by Nintendo, Activision, Beenox,
Demonware, or any original service provider. It contains no proprietary game
code, assets, SDK material, binaries, certificates, keys, firmware, extracted
files, credentials, or raw packet captures. Crash Team Racing Nitro-Fueled is a
trademark of Activision Publishing, Inc.

# Evidence: crash-team-racing

## Static key-check (2026-10-27, base v0)

E0 resolved: base v0 + update v983040 dumps on the pool. Base program NCA
(standard crypto, SDK 6.5.2.0) decompressed and swept:

- Auth: `lavender-switch-auth3.{live,cert,dev}.demonware.net`; lobby:
  `lavender-switch-lobby.{live,cert,dev}.demonware.net` (the `.prod` names in
  older docs resolve to the same IPs — aliases). UNO:
  `https://{prod,cert,dev}.uno.demonware.net/v1.0`.
  ObjectStore + Pipes (AWS glutton) + `stun.{au,eu,jp,us}.demonware.net`.
- TLS confirmed real: NPDM grants `ssl`, NN-SDK libcurl SSL-CTX strings
  present, no bundled CAs (system store via nn::ssl). No OpenSSL/mbedTLS/
  wolfSSL/CyaSSL/GnuTLS/NSS markers anywhere — nn::ssl only. Harbor
  OpenPak-CA injection should MITM E2.
- Env selection is an index (`igNetEnvironment`: dev=0/cert=1/live=2);
  retail uses live. All six hosts registered in one config fill.
- Auth material: `bdAuthTicket` + `bdAuthService`, tasks
  `igNetTaskLsgGetTicket`/`igNetTaskLsgAuthenticateTicket`, console identity
  via `CConsoleId::getHardwareId`. Exact ticket fields need E2/E3.
- Crypto is standard: LibTomCrypt + `bdCryptoUtils` (HKDF, HMAC-SHA1).
- No NEX/Photon/PlayFab. Full notes in workspace research (RECON, private).

## Next experiment

And one cheaper question comes before all of it. OpenPak redirects *Nintendo*
hostnames; it does not redirect `*.demonware.net`. An OpenPak console still
reaches the real, still-running Demonware. **If Demonware accepts it, this title
already works and this repository should close.** That experiment
([E1](evidence-needed.md#e1--does-real-demonware-accept-an-openpak-console))
is one afternoon and it is unrun.

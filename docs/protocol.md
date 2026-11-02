# Protocol: crash-team-racing

## Backend and hosts

**It is a Demonware title, not a NEX title.** The Switch client authenticates
against Activision's own backend over TLS on port 443 —
`lavender-switch-auth3.prod.demonware.net` and
`lavender-switch-lobby.prod.demonware.net`, observed live in our own emulator
tree. Nintendo's NEX, NPLN and BAAS carry none of its online traffic, so this
does **not** belong in [`platforms/switch/games/switch-nex`](../../switch-nex) and this repository is the right
shape.

## Static findings (base v0, update v983040)

From the base program NCA (standard crypto, SDK 6.5.2.0):

- Auth: `lavender-switch-auth3.{live,cert,dev}.demonware.net`; lobby:
  `lavender-switch-lobby.{live,cert,dev}.demonware.net` (`.prod` names resolve to the same
  IPs, aliases). UNO: `https://{prod,cert,dev}.uno.demonware.net/v1.0`. Also ObjectStore,
  Pipes (AWS glutton) and `stun.{au,eu,jp,us}.demonware.net`.
- TLS is real: the NPDM grants `ssl`, NN-SDK libcurl SSL-CTX strings are present, no bundled
  CAs (system store via nn::ssl), no OpenSSL/mbedTLS/wolfSSL/GnuTLS/NSS markers. The OpenPak CA
  in the console store is what a TLS-terminating stub needs.
- Environment selection is an index (`igNetEnvironment`: dev=0, cert=1, live=2); retail uses
  live. All six hosts are registered in one config fill.
- Auth material: `bdAuthTicket` + `bdAuthService`, tasks `igNetTaskLsgGetTicket` /
  `igNetTaskLsgAuthenticateTicket`, console identity via `CConsoleId::getHardwareId`. Exact
  ticket fields are not yet known.
- Crypto is standard: LibTomCrypt + `bdCryptoUtils` (HKDF, HMAC-SHA1).
- No NEX, Photon or PlayFab.

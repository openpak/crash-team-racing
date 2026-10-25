# Protocol: crash-team-racing

## Backend and hosts

**It is a Demonware title, not a NEX title.** The Switch client authenticates
against Activision's own backend over TLS on port 443 —
`lavender-switch-auth3.prod.demonware.net` and
`lavender-switch-lobby.prod.demonware.net`, observed live in our own emulator
tree. Nintendo's NEX, NPLN and BAAS carry none of its online traffic, so this
does **not** belong in [`platforms/switch/games/switch-nex`](../../switch-nex) and this repository is the right
shape.

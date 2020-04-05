# Questions


## Syntax Choice?

`#pragma _STDC embed ...`

`#embed ...`

April 2020 WG16 Vote:
- Unanimous Consent for NO `pragma`!


## Feature Choice?

Type-based
(impl-defined if tokens are not Keywords):
- `#embed keyword-tokens [limit] <header.name>`

Bit-based:
- `#embed bits-per-element [limit] <header.name>`

April 2020 WG14 Votes:
- 10 for Bits (yes), 4 Abstain, 2 for Type-based (no)

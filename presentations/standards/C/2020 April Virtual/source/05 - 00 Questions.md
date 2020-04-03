# Questions


# Syntax Choice?

`#pragma _STDC embed ...`

`#embed ...`

April 2020 Decision: Unanimous Consent for NO `pragma`!


# Feature Choice?

Type-based, Implementation-defined if tokens are not Keywords (+2 Votes):
- `#embed keyword-tokens [limit] <header.name>`

Bit-based (+10 Votes):
- `#embed bits-constant-expression [limit] <header.name>`

April 2020 Decision: Prefer Bit-based approach!

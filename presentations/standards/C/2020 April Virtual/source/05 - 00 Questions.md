# Questions


# Syntax Choice?

`#pragma _STDC embed ...`

`#embed ...`


# Feature Choice?

Basic:
- `#embed [limit] <header.name>`

Type-based, Limited to Keyword Tokens:
- `#embed keyword-tokens [limit] <header.name>`

Type-based, Implementation-defined if tokens are not Keywords:
- `#embed keyword-tokens [limit] <header.name>`

Bit-based:
- `#embed bits-constant-expression [limit] <header.name>`

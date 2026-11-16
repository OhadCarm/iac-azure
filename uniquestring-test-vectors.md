# ARM `uniqueString` — spec & test vectors

Ground-truth input→output pairs captured from real Azure ARM. Use these to validate a reimplementation of `uniqueString`.

## Behavior spec

- **Pure & deterministic:** output depends only on the input string(s) — no subscription/RG/tenant/time. Same input → same output everywhere.
- **Multiple args are joined with `-` then hashed:** `uniqueString(a, b, c)` ≡ `uniqueString(a + "-" + b + "-" + c)`.
- **Output:** exactly **13 characters**, from the RFC-4648 **base32 lowercase** alphabet `abcdefghijklmnopqrstuvwxyz234567` (letters `a`–`z` + digits `2`–`7`; never `0`,`1`,`8`,`9`). May start with a digit.
- Consistent with a 64-bit hash encoded as base32 (13 × 5 = 65 bits). The exact hash is undocumented — validate against vectors, don't re-derive.

## Test vectors

| Input | Expected output |
|---|---|
| `hello` | `htpi75jfihg4e` |
| `world` | `iw4p5eg6q6hta` |
| `azure` | `mdoo6wt7duhyg` |
| `contoso-prod` | `igyfcysomfx3k` |
| `ohadca` | `2qja7fsrqdfeo` |

### Multi-arg / separator checks

| Expression | Expected output |
|---|---|
| `uniqueString("a", "b")` | `i3z4v3sp6aooa` |
| `uniqueString("a-b")` | `i3z4v3sp6aooa` (same as above — proves `-` join) |
| `uniqueString("ab")` | `twldla3s3qb3q` (different) |

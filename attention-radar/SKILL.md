---
name: attention-radar
description: 'Deprecated alias, kept for one release after this skill was renamed to digest-composer. It holds no composing logic of its own. Load it only when a person or a saved config names attention-radar explicitly; it then hands off to digest-composer. Never choose it for a request that does not name it, and never treat it as a digest composer when matching skills by role; use digest-composer.'
license: MIT. The repository's LICENSE has complete terms
compatibility: Needs the digest-composer skill from https://github.com/pendo-io/Product-Loop installed alongside it.
metadata:
  author: Pendo Product Operations
  version: "1.1.0"
---

# Deprecated: renamed to digest-composer

This skill was renamed to `digest-composer` in version 1.1.0. This copy exists only so a habit or saved config that uses the old name keeps working for one release. It does no work of its own.

## What to do when invoked

1. **If `digest-composer` is installed**, follow it exactly as if it had been invoked with the same request and the same inputs. When a person invoked it, say once, first: *"attention-radar is now called digest-composer; using that."*
2. **If it isn't installed**, compose nothing. Say the skill was renamed and offer to install `digest-composer` from https://github.com/pendo-io/Product-Loop.

## Boundaries

- Never compose a digest itself and never write anything.
- The `daily-digest` conductor never runs this stub as its composer; it resolves `digest-composer` instead (its `references/daily-run.md` 3c).
- Remove this folder in the next release, once no saved config uses the old name.

# Octodoge website

Static marketing/support/privacy pages for [octodoge.com](https://octodoge.com). Plain HTML/CSS with static assets — no build step, no package manager, and no automated test or lint suite.

## Cursor Cloud specific instructions

- This is a static site. There are no dependencies to install and no build/lint/test tooling; the update script is effectively a no-op.
- Run the dev server from the repo root with `python3 -m http.server 8765`, then browse [http://localhost:8765](http://localhost:8765). `python3` is preinstalled.
- Clean URLs (`/support/`, `/privacy/`) work locally because each lives in its own directory with an `index.html`. In production these clean URLs depend on CloudFront config (see `README.md`), which is not reproduced by the local server.
- `scripts/upload-website` is for production deploy only (requires AWS CLI + credentials). Do not run it during development.

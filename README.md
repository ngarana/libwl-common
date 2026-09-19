# libwl-common — shared core for the qypr desktop suite

Canonical, vendored Wayland protocol XML consumed by both
[`waylaunch`](https://github.com/ngarana/waylaunch) and
[`qypr`](https://github.com/ngarana/lockscreen) (`lockscreen` repo) as a
**git subtree** at `protocols/` — Stage 2, item 1 of the waylaunch
`docs/INTEGRATION.md` consolidation plan. One copy eliminates the drift bug
class (waylaunch once shipped a hand-reduced 151-line layer-shell next to
qypr's full 407-line upstream).

## Contents

| File | Source of truth | Notes |
|---|---|---|
| `protocols/wlr-layer-shell-unstable-v1.xml` | upstream v5 (identical in both repos) | `get_layer_surface` has a `namespace` arg — a C++ keyword; both repos firewall it at codegen (qypr `qypr_gen_protocol`, waylaunch `cmake/fix_scanner_keywords.cmake`) |
| `protocols/wlr-foreign-toplevel-management-unstable-v1.xml` | qypr's copy (newer: `finished` destructor type, `fullscreen since="2"`) | waylaunch's older copy had comment drift and no destructor type |
| `protocols/wlr-screencopy-unstable-v1.xml` | waylaunch's copy (backdrop blur) | unused by qypr (vendored for the union) |
| `protocols/wlr-gamma-control-unstable-v1.xml` | qypr's copy (night light) | unused by waylaunch (vendored for the union) |

## Consuming (both repos already do this)

Only `protocols/` is shared, so consumers track a split branch, not `main`:

```sh
# first time (replaces the local protocols/ directory):
git rm -r protocols
git commit -m "chore: clear protocols/ for libwl-common subtree"
git subtree add --prefix=protocols <libwl-common-remote> protocols-only --squash

# pull updates (re-split here first, then pull the split branch there):
git subtree split -P protocols -b protocols-only   # in libwl-common
git subtree pull --prefix=protocols <libwl-common-remote> protocols-only --squash
```

`<libwl-common-remote>` is currently a local path; point it at the GitHub
remote once the maintainer creates `ngarana/libwl-common`.

Build paths are unchanged: each repo's CMake still globs its own
`protocols/` dir and runs its own `wayland-scanner` rules (including the
`namespace` firewall) — only the *data* is shared. Each repo generates
only the protocols it uses.

## Versioning

Bump protocol files only to a newer upstream revision, never with local
edits (except the two files' already-absorbed history). After any bump,
rebuild + test **both** consumers before pushing.

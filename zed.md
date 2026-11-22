# Zed (aa2kb/zed) security review notes

- Commit scanned: `20c4627017f3821daeab241fbae9faa6e3d15bd2` (2024-09-11), package version 0.154.0.
- Flagged family: `zed---flagged-vulnerabilities.json` (8 active: 5 HIGH, 3 MEDIUM).

## Runs on this commit

- `bc-ef4b927a-4d2f-58da-9e2e-16064ed49ff1` — collab auth/API (crashes PublicRead, billing StopCancellation IDOR, committed secrets).
- `bc-1efdd61a-f70a-5354-bf37-0ecfa2bd0926` — remote/extensions/exec (formatter RCE, extension LSP binary.path, Zip Slip, WASM spawn escape, unsigned auto-update). Branch `aa2kb/findings-memory-and-reporting-29d1`.

## Active findings (do not drop on same-commit rescan)

1. HIGH — Project `.zed/settings.json` `formatter.external` → host RCE on format/format_on_save; guest edit path included.
2. HIGH — Extension `LspSettings::for_worktree` honors `lsp.*.binary.path` from project settings → host RCE; guest edit path included.
3. HIGH — Zip Slip in `node_runtime::extract_zip` used by extension `download_file` (Zip).
4. HIGH — WASM `language_server_command` / `path_from_extension` not confined; extensions unsigned (checksum TODO).
5. HIGH — Unauthenticated `/telemetry/crashes` PublicRead S3 (+ optional Slack).
6. MEDIUM — Auto-update/remote-server download without signature/hash verify.
7. MEDIUM — Billing `StopCancellation` subscription IDOR (no `billing_customer_id` check) behind API token BFF.
8. MEDIUM — Committed `crates/collab/.env.toml` / `postgrest_app.conf` secrets (do not paste values).

## Triage notes (below bar / not reported)

- Built-in LSP adapters often use `ProjectSettings::get_global`, so project-local `lsp.binary` does not apply to those adapters here; extensions using `for_worktree` still do. Upstream worktree trust (≥ 0.218.2-pre) is newer than this tree.
- `async-tar` 0.5.0 skips `..` on unpack; tar install path traversal not validated as exploitable.
- Context servers use `get_global` only in this commit.
- SSH `spawn_process` is local client plumbing; no guest→SSH injection chain validated.
- REPL kernelspec argv is local Jupyter config, not project-settings auto exec.
- No Slack posting tool on these automations.

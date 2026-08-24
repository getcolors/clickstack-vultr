# clickstack-vultr

Desired state for one [ClickStack](https://github.com/getcolors/clickstack)
observability deployment: ClickHouse, MongoDB, the HyperDX OpenTelemetry
collector and UI on a single Vultr instance in Amsterdam, published at
**https://clickstack.bigconfig.online**.

This repository holds configuration, not code. The behaviour lives in the
`package-clickstack-green` Package Skill installed under `.agents/skills/`.

```sh
direnv allow          # once, after cloning
./green build         # render .colors/ — no provider calls, no credentials
./green create --dry-run
./green create        # converge for real
```

## Configuration

`colors.yml` is the only file to edit and holds non-secret values only.
Credentials are `COLORS_PAR_*` variables in the gitignored `.envrc.private`:

| Credential | Variable |
|---|---|
| Vultr API key | `COLORS_PAR_VULTR_API_KEY` |
| Cloudflare API token (`bigconfig.online` zone) | `COLORS_PAR_CLOUDFLARE_API_TOKEN` |
| R2 state backend | `COLORS_PAR_R2_ACCESS_KEY_ID`, `COLORS_PAR_R2_SECRET_ACCESS_KEY` |

`colors.yml` also names `clickstack-admin-email`, the login for the initial
HyperDX team that convergence creates. The admin password, the team's ingestion
key and the HyperDX session secret are all generated on the server and none is
supplied here; read the first two from `/etc/clickstack/admin.env` and
`/etc/clickstack/ingestion.env`.

## SSH access

The deployment owns its keypair at `~/.ssh/clickstack-vultr`, outside this
checkout, and the Vultr account key named `clickstack-vultr` belongs to its
OpenTofu state. Cloning this repository elsewhere does not carry access — copy
the keypair deliberately. See `CLAUDE.md` for the failure modes and their
recovery paths.

Convergence also writes a `~/.ssh/config` block, so `ssh clickstack-vultr`
connects with no address or flags:

```sh
ssh clickstack-vultr 'cd /opt/clickstack && docker compose ps'
```

## Safety

`compute-prevent-destroy: true` guards deletion; lifting it requires a one-run
`COLORS_PAR_COMPUTE_PREVENT_DESTROY=false` override and separate authorization.
Never export `COLORS_PAR_PROFILE`, and never edit or commit `.colors/`.

# CLAUDE.md

## Repository

Desired state for `clickstack-vultr`: the ClickStack observability stack —
ClickHouse, MongoDB, the HyperDX OpenTelemetry collector, and the HyperDX UI —
on one Vultr instance in Amsterdam, published at
`https://clickstack.bigconfig.online` through Cloudflare and Caddy. Behavior
lives in `../clickstack`.

Tracked source is `colors.yml`, toolchain and documentation, the installed
Package Skill, lockfile, and a root launcher copied from its payload.
`.colors/` is generated private state and `.envrc.private` contains
credentials; never read, edit or commit either.

## The machine keypair is not in this checkout

This deployment runs in **keygen mode**: `colors.yml` carries no
`vultr-ssh-keys`, so the package owns the keypair per the workspace SSH Keypair
Standard. It lives at `~/.ssh/clickstack-vultr`(`.pub`) — outside this
repository — and the Vultr account key named `clickstack-vultr` belongs to this
deployment's OpenTofu state.

Consequences:

- Cloning this repository on another workstation does **not** carry machine
  access. Copy `~/.ssh/clickstack-vultr`(`.pub`) deliberately, or `create` will
  refuse rather than regenerate a key that cannot reach the live host.
- Never delete the Vultr key named `clickstack-vultr` while the instance lives.
- Adding `vultr-ssh-keys` to `colors.yml` switches to opt-out mode and would
  orphan the generated key; a live deployment changes modes only by rebuild.

## Commands

```sh
./green build
./green create --dry-run
./green create
./green delete
```

Build and dry-run require no credentials and never touch `~/.ssh`. Never export
`COLORS_PAR_PROFILE`. Keep `compute-prevent-destroy: true`; deletion requires
separate authorization and a one-run `COLORS_PAR_COMPUTE_PREVENT_DESTROY=false`
override.

## Sending telemetry

The ingestion key is generated on the server, not supplied here:

```sh
ssh -i ~/.ssh/clickstack-vultr root@SERVER \
  'sed -n s/HYPERDX_API_KEY=//p /etc/clickstack/ingestion.env'
```

Point any OTLP/HTTP exporter at `https://clickstack.bigconfig.online` with that
value as its `authorization` header.

## Git

The root `green` is a copy, not a symlink. After a Package Skill update run
`npx skills update -p -y` and copy
`.agents/skills/package-clickstack-green/green` over it. Never hand-edit its
SHA.

Work on the current branch. Do not commit or push unless explicitly asked.
